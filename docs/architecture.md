# Architecture

Skyvern automates browser workflows by combining Playwright (for browser control) with vision-capable LLMs (for understanding what's on screen). This page explains how those pieces fit together and what happens inside the system when you run a task.

## Components at a glance

```
┌─────────────────────────────────────────────────────────────┐
│                     User Interfaces                          │
├───────────────────────┬──────────────────────┬──────────────┤
│  Web UI (React)       │  Python SDK / CLI    │  TypeScript  │
│  localhost:8080       │  skyvern library     │  SDK         │
└───────────────────────┴──────────────────────┴──────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────┐
│                  FastAPI HTTP API  (port 8000)               │
│             forge/api_app.py + forge/sdk/routes/            │
└──────────────────────┬──────────────────────────────────────┘
                       │
            ┌──────────┴──────────────────────┐
            │                                 │
            ▼                                 ▼
   ┌──────────────────┐             ┌────────────────────┐
   │   Agent Core     │             │  Workflow Engine    │
   │  forge/agent.py  │             │  workflow/service  │
   │                  │             │                    │
   │  Step generation │             │  Block execution   │
   │  LLM calling     │             │  Parameter passing │
   │  Vision analysis │             │  Loop/branch logic │
   │  Action dispatch │             │                    │
   └────────┬─────────┘             └──────────┬─────────┘
            │                                  │
            └──────────────┬───────────────────┘
                           │
           ┌───────────────┼───────────────┐
           ▼               ▼               ▼
       ┌────────┐   ┌────────────┐   ┌──────────┐
       │Browser │   │  LLM APIs  │   │PostgreSQL│
       │(Play-  │   │ via LiteLLM│   │          │
       │wright) │   │            │   │          │
       └────────┘   └────────────┘   └──────────┘
```

The API layer accepts requests and routes them to either the **Agent Core** (for direct tasks) or the **Workflow Engine** (for multi-block workflows). Both ultimately drive the same browser and LLM infrastructure, with results persisted to PostgreSQL.

---

## The ForgeApp: one shared runtime

All services share a single `ForgeApp` instance defined in `forge/forge_app.py`. It holds references to every major subsystem: the database, storage backend, artifact manager, browser manager, LLM handlers, and workflow service. Think of it as a dependency-injection container that initializes once when the server starts.

```python
# forge/__init__.py
app: ForgeApp  # global singleton
```

Anywhere in the codebase that needs, say, the database or an LLM handler, imports `from skyvern.forge import app` and accesses `app.DATABASE` or `app.LLM_API_HANDLER`.

---

## How a task executes

When you `POST /api/v1/run/tasks`, here is what happens step by step:

```
1. API receives TaskRunRequest
   ↓
2. Task record written to PostgreSQL (status = QUEUED)
   ↓
3. Agent picks up the task, status → RUNNING
   ↓
4. Browser navigates to the starting URL
   ↓
5. Agent takes a screenshot
   ↓
6. Screenshot + page context sent to the vision LLM
   ↓
7. LLM responds with an action (click, type, scroll, extract, etc.)
   ↓
8. Playwright executes the action
   ↓
9. Step record written to DB (tokens used, action taken, result)
   ↓
10. Loop back to step 5 until:
     - completion criterion met → COMPLETED
     - error or max steps reached → FAILED
     - user cancels → TERMINATED
    ↓
11. Webhook fired (if webhook_url was provided)
```

The agent doesn't pre-plan the whole task. It reasons one screenshot at a time, which is how it can handle pages it has never seen before — there are no hardcoded selectors to break. Each loop iteration is a **Step**, persisted individually so you can inspect exactly what the agent did and why.

---

## How a workflow executes

A workflow is a sequence of **blocks**. The `WorkflowService` iterates through them in order, passing outputs from one block as inputs to the next.

```
WorkflowRun created with input parameters
         ↓
   For each block in sequence:
         ↓
   ┌──────────────────────────────────────────────┐
   │  ActionBlock     → creates a Task,            │
   │                    waits for completion       │
   │  ValidationBlock → evaluates a criterion,     │
   │                    may branch or stop early   │
   │  ForLoopBlock    → runs child blocks N times  │
   │  TextPromptBlock → calls LLM directly,        │
   │                    stores response as output  │
   │  HttpRequestBlock → makes an HTTP call        │
   │  NavigationBlock, LoginBlock, ExtractionBlock │
   │  CodeBlock, SendEmailBlock, FileDownloadBlock │
   │  ConditionalBlock, WaitBlock, ... (25+ types) │
   └──────────────────────────────────────────────┘
         ↓
   WorkflowRun completed, webhook fired
```

Blocks share state through a `WorkflowRunContext`. When an `ActionBlock` finishes, its extracted output is stored in the context under a named `OutputParameter`. Downstream blocks can reference it with Jinja-style template syntax like `{{ output_1.product_price }}`.

---

## Key data models

### Task

A task is the fundamental unit of work, stored in the `tasks` table.

| Field | Purpose |
|---|---|
| `task_id` | Unique identifier |
| `url` | Starting URL |
| `navigation_goal` | What to achieve (natural language) |
| `data_extraction_goal` | What data to pull out |
| `navigation_payload` | Contextual input (form values, credentials, etc.) |
| `extracted_information` | The final output, as JSON |
| `status` | `QUEUED` → `RUNNING` → `COMPLETED` / `FAILED` / `TERMINATED` |

### Step

Each action the agent takes is a step. Steps belong to a task and are stored in the `steps` table.

| Field | Purpose |
|---|---|
| `step_id` | Unique identifier |
| `order` | Sequential position within the task |
| `output` | Action taken and its result |
| `input_token_count` | LLM tokens consumed on input |
| `output_token_count` | LLM tokens consumed on output |
| `status` | `SUCCESS` or `FAILED` |

### Workflow and WorkflowRun

A `Workflow` is the template — it stores block definitions and a parameter schema. A `WorkflowRun` is one execution of that template with specific inputs. The same workflow can run many times with different parameters, and each run is tracked independently.

### Artifact

Every screenshot, HTML snapshot, downloaded file, or extracted CSV is an `Artifact`. Artifacts are stored in S3, Azure Blob, or the local filesystem depending on your `SKYVERN_STORAGE_TYPE` setting, with their URIs persisted in the `artifacts` table.

---

## LLM routing

Skyvern uses **LiteLLM** as a unified interface to every LLM provider. The `LLMConfigRegistry` maps named keys (like `OPENAI_GPT4O`, `ANTHROPIC_CLAUDE3.5_SONNET`) to provider-specific configurations. The active model is chosen via the `LLM_KEY` environment variable.

`ForgeApp` holds multiple specialized LLM handlers beyond the primary one. For example, `SELECT_AGENT_LLM_KEY` controls which model handles element selection, and `EXTRACTION_LLM_KEY` controls data extraction. Each can point to a different model, letting you balance cost and capability per task type.

---

## Storage backends

Artifacts can be stored in three places:

- **Local filesystem** (`SKYVERN_STORAGE_TYPE=local`) — default for development
- **AWS S3** (`SKYVERN_STORAGE_TYPE=s3cloud`) — recommended for production
- **Azure Blob Storage** (`SKYVERN_STORAGE_TYPE=azureblob`) — for Azure deployments

The storage backend is abstracted behind a `BaseStorage` interface, so switching is a configuration change with no code changes.

---

## Multi-tenancy

Every database record carries an `organization_id`. The API layer extracts org context from the API key on each request and scopes all database queries to that org. A single Skyvern instance can serve multiple isolated tenants without any data leaking between them.

---

## Design tradeoffs

**Why screenshot-by-screenshot instead of DOM parsing?**

Traditional automation relies on XPath or CSS selectors, which break whenever a site changes its HTML structure. Skyvern's vision approach treats the page as an image, the way a human does. The tradeoff is higher LLM cost per step and slower execution, but far greater resilience to UI changes.

**Why steps instead of a single LLM call?**

Complex tasks require many actions. Breaking them into steps lets the agent observe the result of each action before deciding what to do next. It also makes debugging tractable — you can look at step 4's screenshot and see exactly what the agent saw when it made a wrong decision.

**Why PostgreSQL instead of an in-memory store?**

Persisting every step means tasks survive server restarts, can be inspected after the fact, and can be distributed across multiple agent processes if you scale horizontally.
