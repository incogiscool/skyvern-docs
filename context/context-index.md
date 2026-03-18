# Context Index: Skyvern

## Overview

Skyvern is an open-source browser automation platform that uses large language models (LLMs) and computer vision to automate browser-based workflows — eliminating the need for hand-written CSS selectors or XPath. Instead of brittle DOM queries, Skyvern's agent analyses live screenshots, reasons about page structure, and issues Playwright actions autonomously.

The platform ships as four integrated surfaces: a **Python SDK** (Playwright-compatible with AI capabilities), a **TypeScript SDK**, a **REST/WebSocket API** (FastAPI backend), and a **web UI** (React/Vite) for building and monitoring workflows. Tasks and workflows are persisted in PostgreSQL via SQLAlchemy async ORM, and browser sessions are managed through Playwright on Chromium.

Skyvern supports multiple LLM providers (OpenAI, Anthropic/Claude, Google Vertex AI) through LiteLLM, making it possible to swap providers via environment variables. It is designed for multi-tenant SaaS use: every task, workflow, credential, and session is scoped to an `organization_id` enforced at both the API and database layers.

Primary use cases include automated form filling, data extraction, login flows, and multi-step web workflows that would otherwise require manual effort or fragile DOM scripting.

**Core Stack**: Python 3.11+, FastAPI, SQLAlchemy (async), Playwright, LiteLLM, Pydantic v2, PostgreSQL, Alembic, React 18, TypeScript, Vite

---

## File Tree

```
context/skyvern/
├── skyvern/                          # Main Python package
│   ├── __init__.py                   # Lazy import exports (Skyvern, SkyvernPage, workflow, action, etc.)
│   ├── __main__.py                   # CLI entry point
│   ├── _cli_bootstrap.py             # CLI initialization
│   ├── config.py                     # Settings and configuration (L24+: Settings class, env variables)
│   ├── constants.py                  # App constants
│   ├── exceptions.py                 # HTTP and domain exceptions
│   ├── analytics.py                  # Analytics tracking
│   │
│   ├── cli/                          # Command-line interface
│   │   ├── __init__.py
│   │   ├── auth_command.py
│   │   ├── browser.py
│   │   ├── commands/                 # Command submodules
│   │   ├── core/                     # CLI utilities: browser launcher, client, session manager, auth
│   │   ├── mcp_tools/                # Model Context Protocol tools (browser, session, blocks, credentials, etc.)
│   │   ├── quickstart.py             # Quickstart command
│   │   ├── run_commands.py           # Task/workflow execution commands
│   │   ├── setup_commands.py         # Setup commands
│   │   ├── workflow.py               # Workflow commands
│   │   └── [other setup/config commands]
│   │
│   ├── forge/                        # Core agent and API infrastructure
│   │   ├── __init__.py               # Global forge app instance
│   │   ├── agent.py                  # Main Agent class: orchestrates task execution, step generation, LLM calls
│   │   ├── agent_functions.py        # Helper functions for agent actions
│   │   ├── api_app.py                # FastAPI app setup, middleware, lifespan
│   │   ├── async_operations.py       # Async operation pool for concurrent execution
│   │   ├── forge_app.py              # ForgeApp class: main runtime configuration
│   │   ├── forge_openai_client.py    # OpenAI client wrapper
│   │   ├── prompts.py                # Prompt engineering templates
│   │   ├── request_logging.py        # Request/response logging middleware
│   │   │
│   │   ├── sdk/                      # Forge SDK
│   │   │   ├── api/                  # API integrations
│   │   │   │   ├── llm/              # LLM provider support (OpenAI, Anthropic, Vertex, etc.)
│   │   │   │   ├── aws.py, azure.py  # Cloud provider APIs
│   │   │   │   ├── crypto.py         # Cryptographic utilities
│   │   │   │   ├── email.py          # Email handling
│   │   │   │   └── files.py          # File operations
│   │   │   │
│   │   │   ├── artifact/             # Artifact storage management
│   │   │   │   ├── manager.py        # Artifact manager (bulk creation, retrieval)
│   │   │   │   ├── models.py         # Artifact data models
│   │   │   │   └── storage/          # Storage backends (Azure, S3, local)
│   │   │   │
│   │   │   ├── core/                 # Core utilities
│   │   │   │   ├── skyvern_context.py # SkyvernContext: request context management
│   │   │   │   ├── security.py       # JWT and webhook signature generation
│   │   │   │   └── [other utilities]
│   │   │   │
│   │   │   ├── db/                   # Database abstraction
│   │   │   │   ├── models.py         # SQLAlchemy ORM models (TaskModel, StepModel, WorkflowModel, etc.)
│   │   │   │   ├── agent_db.py       # Database CRUD operations for agent
│   │   │   │   ├── base_alchemy_db.py # Base database layer
│   │   │   │   ├── enums.py          # Database enums (TaskStatus, StepStatus, etc.)
│   │   │   │   └── exceptions.py     # Database exceptions
│   │   │   │
│   │   │   ├── models.py             # Domain models (Step, StepStatus, SpeculativeLLMMetadata, etc.)
│   │   │   │
│   │   │   ├── routes/               # API endpoints/routers
│   │   │   │   ├── routers.py        # Main router composition
│   │   │   │   ├── agent_protocol.py # Agent protocol endpoints
│   │   │   │   ├── browser_profiles.py
│   │   │   │   ├── browser_sessions.py
│   │   │   │   ├── credentials.py
│   │   │   │   ├── debug_sessions.py
│   │   │   │   ├── prompts.py
│   │   │   │   ├── run_blocks.py
│   │   │   │   ├── scripts.py        # Script API endpoints
│   │   │   │   ├── streaming/        # WebSocket and SSE streaming
│   │   │   │   ├── webhooks.py
│   │   │   │   └── [other endpoints]
│   │   │   │
│   │   │   ├── schemas/              # Request/response schemas
│   │   │   │   ├── tasks.py          # Task request/response schemas
│   │   │   │   ├── workflows.py      # Workflow schemas
│   │   │   │   └── [other domain schemas]
│   │   │   │
│   │   │   └── workflow/             # Workflow orchestration
│   │   │       ├── models/           # Workflow block models (ActionBlock, ValidationBlock, etc.)
│   │   │       ├── context_manager.py # WorkflowRunContext
│   │   │       ├── service.py        # Workflow execution service
│   │   │       └── workflow_definition_converter.py
│   │   │
│   │   └── [other forge modules]
│   │
│   ├── library/                      # High-level SDK API
│   │   ├── skyvern.py               # Skyvern class: main library interface
│   │   ├── skyvern_browser.py       # SkyvernBrowser class
│   │   ├── skyvern_browser_page.py  # SkyvernBrowserPage
│   │   ├── skyvern_browser_page_agent.py    # Browser page agent logic
│   │   ├── skyvern_browser_page_ai.py       # AI interactions (locators, element finding)
│   │   ├── ai_locator.py            # AI-powered element location
│   │   └── skyvern_locator.py
│   │
│   ├── core/                         # Core features (referenced via lazy imports)
│   │   ├── script_generations/       # Script generation and wrappers
│   │   │   ├── skyvern_page.py      # SkyvernPage, RunContext classes
│   │   │   ├── workflow_wrappers.py # @workflow and @cached decorators
│   │   │   └── run_initializer.py   # setup() function
│   │   └── [other core modules]
│   │
│   ├── services/                     # Business logic services
│   │   ├── action_service.py         # Action execution logic
│   │   ├── block_service.py
│   │   ├── browser_recording/        # Browser recording state machines
│   │   ├── browser_session_service.py
│   │   ├── error_detection_service.py
│   │   ├── otp_service.py
│   │   ├── pdf_import_service.py
│   │   ├── run_code_service.py
│   │   ├── run_service.py
│   │   ├── script_service.py         # Script action functions (action, download, extract, goto, etc.)
│   │   ├── task_v1_service.py        # V1 task service
│   │   ├── task_v2_service.py        # V2 task service (new architecture)
│   │   ├── workflow_service.py
│   │   └── webhook_service.py
│   │
│   ├── schemas/                      # Pydantic schema definitions
│   │   ├── artifacts.py
│   │   ├── browser_sessions.py
│   │   ├── folders.py
│   │   ├── runs.py
│   │   ├── scripts.py
│   │   ├── steps.py
│   │   ├── webhooks.py
│   │   └── workflows.py
│   │
│   ├── webeye/                       # Vision-based web interaction
│   │   └── [vision modules]
│   │
│   ├── experimentation/              # Feature flags and experiments
│   ├── library/                      # [as above]
│   └── utils/                        # Utility functions
│
├── skyvern-frontend/                # React/TypeScript web UI
│   ├── src/
│   │   ├── api/                     # API client calls
│   │   ├── components/              # Reusable React components
│   │   ├── hooks/                   # Custom React hooks
│   │   ├── routes/                  # Page components and route structure
│   │   │   ├── tasks/               # Task management UI
│   │   │   ├── workflows/           # Workflow builder UI
│   │   │   ├── browserSessions/     # Browser session management
│   │   │   ├── credentials/         # Credentials management
│   │   │   ├── history/             # Execution history
│   │   │   ├── runs/                # Task/workflow run monitoring
│   │   │   └── settings/            # Application settings
│   │   └── assets/
│   ├── package.json
│   ├── vite.config.ts
│   └── tailwind.config.js
│
├── skyvern-ts/                      # TypeScript SDK
│   └── client/
│
├── alembic/                         # Database migrations
│   └── versions/                    # Migration files
│
├── tests/                           # Test suite
│   ├── unit/                        # Unit tests
│   ├── sdk/                         # SDK tests
│   └── smoke_tests/                 # Integration tests
│
├── pyproject.toml                   # Python project configuration
├── docker-compose.yml               # Docker Compose setup
├── Dockerfile                       # Backend Docker image
├── Dockerfile.ui                    # Frontend Docker image
├── README.md
└── [other config files]
```

---

## Architecture

### System Design

Skyvern uses a **multi-component architecture** for browser automation:

```
┌─────────────────────────────────────────────────────────────┐
│                     User Interfaces                          │
├──────────────────────┬──────────────────────┬────────────────┤
│  Web UI (React)      │  Python SDK/CLI      │  TypeScript SDK│
│  (skyvern-frontend)  │  (skyvern library)   │  (skyvern-ts)  │
└──────────────────────┬──────────────────────┬────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                  FastAPI HTTP API                           │
│                (forge/api_app.py)                           │
├─────────────────────────────────────────────────────────────┤
│  Routers: tasks, workflows, scripts, browser_sessions,    │
│           credentials, run_blocks, streaming, webhooks     │
└──────────────┬──────────────────────────────────────────────┘
               │
        ┌──────┴──────────────────────────────┐
        │                                     │
        ▼                                     ▼
   ┌─────────────────────┐          ┌─────────────────────┐
   │  Agent Core         │          │  Workflow Engine    │
   │  (forge/agent.py)   │          │  (workflow/service) │
   │                     │          │                     │
   │ - Step generation   │          │ - Block execution   │
   │ - LLM calling       │          │ - Parameter passing │
   │ - Vision analysis   │          │ - Loop/condition    │
   │ - Action execution  │          │   handling          │
   └─────────┬───────────┘          └──────────┬──────────┘
             │                                 │
             └──────────┬──────────────────────┘
                        │
        ┌───────────────┼───────────────┐
        ▼               ▼               ▼
    ┌────────┐  ┌──────────────┐  ┌──────────┐
    │Browser │  │  LLM API     │  │Database  │
    │(Play-  │  │  (via Lite   │  │(Postgres)│
    │wright) │  │   LLM)       │  │          │
    └────────┘  └──────────────┘  └──────────┘
```

### Key Components

1. **Agent** (`forge/agent.py`)
   - Main orchestrator for task execution
   - Manages task lifecycle (queued → running → completed)
   - Generates steps iteratively by calling LLMs
   - Executes actions on the browser
   - Handles error detection and retries

2. **Services** (`services/`)
   - **action_service.py**: Converts LLM actions to Playwright operations
   - **script_service.py**: Script action decorators (@action, @workflow, etc.)
   - **browser_session_service.py**: Manages browser instances
   - **webhook_service.py**: Handles webhooks for task completion
   - **workflow_service.py**: High-level workflow execution

3. **Library** (`library/`)
   - **Skyvern class**: Main entry point for programmatic use
   - **SkyvernBrowser**: Browser wrapper with AI capabilities
   - **SkyvernBrowserPage**: Page wrapper with AI locators

4. **API** (`forge/api_app.py` + `forge/sdk/routes/`)
   - FastAPI application with CORS, error handling, request logging
   - RESTful endpoints for tasks, workflows, scripts, credentials, browser sessions
   - WebSocket/SSE streaming for real-time execution feedback

5. **Database** (`forge/sdk/db/`)
   - SQLAlchemy ORM models for Tasks, Steps, Workflows, RunBlocks, etc.
   - Async database operations via asyncpg
   - Alembic migrations for schema versioning

6. **Frontend** (`skyvern-frontend/`)
   - React SPA for task/workflow management
   - Real-time monitoring and control via API integration

### Data Flow: Task Execution

```
1. User submits task via API (POST /tasks)
   ↓
2. Task stored in DB with status=QUEUED
   ↓
3. Agent picks up task, updates status to RUNNING
   ↓
4. Agent takes screenshot, analyzes with vision LLM
   ↓
5. LLM returns action (click, type, scroll, etc.)
   ↓
6. Action executed via Playwright
   ↓
7. Step stored in DB with action result, tokens used, etc.
   ↓
8. Repeat steps 4-7 until: completion criterion met, error, or max steps reached
   ↓
9. Task status updated to COMPLETED, FAILED, or TERMINATED
   ↓
10. Result sent via webhook (if provided)
    ↓
11. User retrieves result via GET /tasks/{task_id}
```

### Workflow Execution

Workflows are defined as DAGs of blocks (ActionBlock, ValidationBlock, LoopBlock, etc.). The **WorkflowService** orchestrates execution:

```
1. User submits workflow run
   ↓
2. WorkflowRunContext created with parameters
   ↓
3. Iterate through blocks in order
   ↓
4. For ActionBlock: Create task, wait for completion
   ↓
5. For ValidationBlock: Check criterion, branch if needed
   ↓
6. For LoopBlock: Execute child blocks repeatedly
   ↓
7. Upon completion: Save WorkflowRun, emit webhook
```

---

## Key Concepts

### Task
- **Defined in**: `forge/sdk/schemas/tasks.py`, `forge/sdk/db/models.py:TaskModel`
- **Describes**: A unit of browser automation work
  - `url`: Starting URL
  - `navigation_goal`: What to achieve
  - `data_extraction_goal`: What data to extract
  - `navigation_payload`: Contextual data (form fields, etc.)
  - `extracted_information_schema`: Expected JSON schema of output
- **Lifecycle**: QUEUED → RUNNING → COMPLETED/FAILED/TERMINATED
- **Storage**: PostgreSQL `tasks` table

### Step
- **Defined in**: `forge/sdk/models.py`, `forge/sdk/db/models.py:StepModel`
- **Describes**: An individual action taken during task execution
  - `order`: Sequential step number
  - `action`: Type of action (e.g., click, type, scroll)
  - `output`: Result of the action (e.g., new page state, extracted text)
  - `status`: SUCCESS, FAILED, etc.
  - `input_token_count`, `output_token_count`: LLM token usage
- **Used by**: Agent to track progress and inform future decisions

### Workflow
- **Defined in**: `forge/sdk/schemas/workflows.py`, `forge/sdk/workflow/models/workflow.py`
- **Describes**: A reusable automation template with parameters
  - `permanent_id`: Unique identifier (can have multiple versions)
  - `version`: Version number
  - `blocks`: Array of ActionBlock, ValidationBlock, LoopBlock, etc.
  - `parameters`: Input parameters (name, type, required)
- **Used for**: Repeatable multi-step workflows (form submission with multiple pages, etc.)

### Artifact
- **Defined in**: `forge/sdk/artifact/models.py`, `forge/sdk/db/models.py:ArtifactModel`
- **Describes**: A file or media produced during execution
  - `type`: screenshot, html, pdf, csv, etc.
  - `uri`: Storage path (S3, Azure, or local)
  - `created_at`: When it was created
- **Used for**: Evidence collection, debugging, data extraction results

### Block
- **Defined in**: `forge/sdk/workflow/models/block.py`
- **Types**:
  - **ActionBlock**: Execute a task step
  - **ValidationBlock**: Check a condition, branch on result
  - **LoopBlock**: Repeat child blocks
  - **BaseTaskBlock**: Generic task definition
- **Used for**: Composing complex workflows from simple building blocks

### BrowserSession
- **Defined in**: `forge/sdk/schemas/browser_sessions.py`, `forge/sdk/db/models.py:BrowserSessionModel`
- **Describes**: A persistent browser instance
  - `browser_session_id`: Unique identifier
  - `cdp_url`: Chrome DevTools Protocol endpoint
  - `status`: RUNNING, IDLE, COMPLETED
  - `persistent`: Whether to keep alive between tasks
- **Used for**: Maintaining state across multiple tasks (logged-in sessions, etc.)

---

## API Surface

### Main Endpoints

#### Tasks
- **POST /tasks** — Create and start a task
  - Request: `TaskRequest` (url, navigation_goal, data_extraction_goal, navigation_payload, etc.)
  - Response: `TaskResponse` (task_id, status, extracted_information, etc.)
- **GET /tasks/{task_id}** — Get task status and results
- **GET /tasks** — List tasks (with pagination, filters)
- **POST /tasks/{task_id}/cancel** — Cancel a running task
- **Location**: `forge/sdk/routes/agent_protocol.py`

#### Workflows
- **POST /workflows** — Create a workflow
- **GET /workflows/{workflow_id}** — Get workflow details
- **PUT /workflows/{workflow_id}** — Update a workflow
- **POST /workflows/{workflow_id}/runs** — Start a workflow run
- **GET /workflow-runs/{run_id}** — Get workflow run status
- **Location**: `forge/sdk/routes/[TODO: find exact files]`

#### Browser Sessions
- **POST /browser-sessions** — Create a new browser session
- **GET /browser-sessions/{browser_session_id}** — Get session details
- **WebSocket /browser-sessions/{browser_session_id}/stream** — Real-time browser control
- **Location**: `forge/sdk/routes/browser_sessions.py`

#### Credentials
- **POST /credentials** — Store a credential
- **GET /credentials** — List credentials
- **DELETE /credentials/{credential_id}** — Remove a credential
- **Location**: `forge/sdk/routes/credentials.py`

#### Scripts (SDK-based execution)
- **POST /scripts** — Create and run a script
- **GET /scripts/{script_id}** — Get script results
- **Location**: `forge/sdk/routes/scripts.py`

#### Run Blocks (Workflow blocks)
- **POST /workflow-runs/{run_id}/blocks** — Start a block
- **GET /workflow-runs/{run_id}/blocks/{block_id}** — Get block status
- **Location**: `forge/sdk/routes/run_blocks.py`

#### Streaming
- **WebSocket /ws** — Real-time event streaming (screenshots, actions, etc.)
- **SSE /events** — Server-Sent Events for execution updates
- **Location**: `forge/sdk/routes/streaming/`

#### Webhooks
- **POST /webhooks** — Register a webhook callback
- **Location**: `forge/sdk/routes/webhooks.py`

---

## Data Model

### Core Database Models

1. **TaskModel** (`forge/sdk/db/models.py`)
   ```
   task_id: String (PK)
   organization_id: String (FK → organizations)
   status: String (QUEUED, RUNNING, COMPLETED, etc.)
   url, navigation_goal, data_extraction_goal: String
   navigation_payload: JSON
   extracted_information: JSON
   extracted_information_schema: JSON
   webhook_callback_url: String
   proxy_location: String
   workflow_run_id: String (FK → workflow_runs, nullable)
   browser_session_id: String (FK → browser_sessions, nullable)
   created_at, modified_at, started_at, finished_at: DateTime
   ```

2. **StepModel** (`forge/sdk/db/models.py`)
   ```
   step_id: String (PK)
   task_id: String (FK → tasks)
   status: String (SUCCESS, FAILED, etc.)
   output: JSON
   order: Integer
   input_token_count, output_token_count, reasoning_token_count: Integer
   created_at, modified_at: DateTime
   ```

3. **WorkflowModel** (`forge/sdk/db/models.py`)
   ```
   workflow_id: String (PK)
   permanent_id: String (unique per workflow, can have multiple versions)
   version: Integer
   title: String
   description: String
   definition: JSON (serialized block definitions)
   parameters: JSON (parameter schema)
   created_at, modified_at: DateTime
   ```

4. **WorkflowRunModel** (`forge/sdk/db/models.py`)
   ```
   workflow_run_id: String (PK)
   workflow_id: String (FK → workflows)
   status: String (QUEUED, RUNNING, COMPLETED, etc.)
   inputs: JSON (provided parameter values)
   output: JSON (final results)
   webhook_callback_url: String (nullable)
   created_at, modified_at: DateTime
   ```

5. **BrowserSessionModel** (`forge/sdk/db/models.py`)
   ```
   browser_session_id: String (PK)
   organization_id: String (FK)
   cdp_url: String (Chrome DevTools Protocol endpoint)
   status: String (RUNNING, IDLE, COMPLETED)
   persistent: Boolean
   created_at, modified_at: DateTime
   ```

6. **ArtifactModel** (`forge/sdk/db/models.py`)
   ```
   artifact_id: String (PK)
   type: String (screenshot, html, pdf, csv, etc.)
   uri: String (storage location)
   task_id, step_id, workflow_run_id: String (FK, nullable)
   created_at: DateTime
   ```

7. **CredentialModel** (`forge/sdk/db/models.py`)
   ```
   credential_id: String (PK)
   organization_id: String (FK)
   type: String (api_key, username_password, oauth, etc.)
   value: String (encrypted)
   created_at: DateTime
   ```

### Pydantic Request/Response Schemas

- **TaskRequest**, **TaskResponse** — `forge/sdk/schemas/tasks.py`
- **WorkflowRequest**, **WorkflowResponse** — `forge/sdk/schemas/workflows.py`
- **BrowserSessionRequest**, **BrowserSessionResponse** — `forge/sdk/schemas/browser_sessions.py`
- **StepResponse**, **StepsResponse** — `forge/sdk/schemas/steps.py`
- **CredentialResponse** — `forge/sdk/schemas/credentials.py` [inferred]

### Enums

- **TaskStatus** (`forge/sdk/db/enums.py`): QUEUED, RUNNING, COMPLETED, FAILED, TERMINATED, etc.
- **StepStatus** (`forge/sdk/models.py`): SUCCESS, FAILED, RETRY, etc.
- **TaskType** (`forge/sdk/db/enums.py`): general, validation, extract, etc.
- **ProxyLocationInput** (`forge/sdk/schemas/tasks.py`): US, EU, UK, CA, IN, JP, GB, etc.
- **ThoughtType** (`forge/sdk/schemas/task_v2.py`): identify_action, validate_action, etc.

---

## Auth & Middleware

### Authentication

1. **JWT-based Authentication** (`forge/sdk/core/security.py`)
   - Uses PyJWT library with HS256 algorithm by default
   - Secret key: `SECRET_KEY` environment variable (PLACEHOLDER in dev)
   - Token expiration: `ACCESS_TOKEN_EXPIRE_MINUTES` (default: 7 days)

2. **Internal Auth Middleware** (`forge/sdk/routes/internal_auth.py`)
   - Validates API tokens in headers
   - Extracts organization context

### Middleware Stack (`forge/api_app.py`)

1. **RawContextMiddleware** — Adds request context (execution date, headers)
2. **CORSMiddleware** — Configurable CORS headers (default: `["*"]`)
3. **Error Handler Middleware**:
   - Catches `SkyvernHTTPException` and formats response
   - Catches `NotFoundError` → 404
   - Catches `ValidationError` → 400 with formatted message
4. **Request Logging Middleware** (`request_logging.py`)
   - Logs raw requests/responses (configurable via `LOG_RAW_API_REQUESTS`)
   - JSON logging support (`JSON_LOGGING` flag)

---

## Configuration

| Variable | Purpose | Default | File |
|----------|---------|---------|------|
| `ENV` | Deployment environment (local, staging, prod) | "local" | config.py:L70 |
| `PORT` | HTTP server port | 8000 | config.py:L71 |
| `SECRET_KEY` | JWT signing key | "PLACEHOLDER" | config.py:L64 |
| `SIGNATURE_ALGORITHM` | JWT algorithm | "HS256" | config.py:L65 |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | Token TTL | 10080 (7 days) | config.py:L66 |
| `DATABASE_STRING` | PostgreSQL connection URL | platform-dependent | config.py:L59-61 |
| `DATABASE_REPLICA_STRING` | Read replica URL (optional) | None | config.py:L62 |
| `DATABASE_POOL_SIZE` | SQLAlchemy connection pool size | 5 | config.py:L64 |
| `MAX_STEPS_PER_RUN` | Max steps per task execution | 10 | config.py:L52 |
| `MAX_STEPS_PER_TASK_V2` | Max steps for TaskV2 | 25 | config.py:L53 |
| `MAX_RETRIES_PER_STEP` | Max retry attempts per step | 5 | config.py:L58 |
| `BROWSER_TYPE` | Browser choice (chromium, firefox, etc.) | "chromium-headful" | config.py:L36 |
| `BROWSER_REMOTE_DEBUGGING_URL` | CDP endpoint | "http://127.0.0.1:9222" | config.py:L37 |
| `CHROME_EXECUTABLE_PATH` | Path to Chrome binary (optional) | None | config.py:L38 |
| `VIDEO_PATH` | Directory for browser recordings | "./video" | config.py:L40 |
| `HAR_PATH` | Directory for HAR files | "./har" | config.py:L41 |
| `LOG_PATH` | Directory for logs | "./log" | config.py:L42 |
| `DOWNLOAD_PATH` | Directory for downloads | "{REPO_ROOT_DIR}/downloads" | config.py:L44 |
| `BROWSER_SCREENSHOT_TIMEOUT_MS` | Screenshot timeout | 20000 | config.py:L47 |
| `BROWSER_LOADING_TIMEOUT_MS` | Page load timeout | 60000 | config.py:L48 |
| `ENABLE_PROXY` | Enable proxy usage | False | config.py:L78 |
| `HOSTED_PROXY_POOL` | Proxy endpoints (CSV) | "" | config.py:L76 |
| `LOG_LEVEL` | Python logging level | "INFO" | config.py:L73 |
| `JSON_LOGGING` | Structured JSON logging | False | config.py:L72 |
| `DEBUG_MODE` | Debug mode flag | False | config.py:L57 |

**Environment files loaded** (in order): `.env`, `.env.staging`, `.env.prod` — First found wins.

---

## Dependencies

### Python Core
- **fastapi** (≥0.121.0) — Web framework
- **uvicorn[standard]** (≥0.35.0) — ASGI server
- **pydantic**, **pydantic-settings** (≥2.5.2) — Data validation and settings
- **sqlalchemy[mypy]** (≥2.0.29) — ORM
- **alembic** (≥1.12.1) — Database migrations
- **asyncpg** (≥0.30.0), **psycopg[binary,pool]** (3.1.18) — PostgreSQL drivers

### Browser & Vision
- **playwright** (>1.46.0) — Browser automation
- **pillow** (≥10.2.0) — Image processing
- **python-calamine** (≥0.6.1) — Excel parsing

### LLM & AI
- **openai** (≥1.68.2) — OpenAI API
- **litellm** (≥1.80.10) — Multi-LLM provider support
- **anthropic** (≥0.50.0) — Claude API
- **google-cloud-aiplatform** (≥1.90.0) — Vertex AI
- **tiktoken** (≥0.9.0) — Token counting

### Cloud & Storage
- **aioboto3** (≥14.3.0) — AWS SDK (async)
- **azure-storage-blob** (≥12.26.0) — Azure Blob Storage
- **azure-identity**, **azure-keyvault-secrets** — Azure auth
- **stripe** (≥9.7.0) — Payment processing (cloud only)

### Utilities
- **python-dotenv** (≥1.0.0) — .env file loading
- **jinja2** (≥3.1.2) — Templating
- **structlog** (≥23.2.0) — Structured logging
- **typer** (≥0.16.0) — CLI framework
- **cachetools** (≥5.3.2) — Caching
- **aioredlock** (≥0.7.3) — Redis distributed locking
- **python-jose[cryptography]** (≥3.4.0) — JWT
- **httpx[socks]** (≥0.27.0) — HTTP client
- **websockets** (≥12.0) — WebSocket support
- **pyotp** (≥2.9.0) — OTP/TOTP codes

### Testing (dev)
- **pytest**, **pytest-asyncio** — Test framework
- **moto[s3,server]** (≥5.1.5) — AWS mocking
- **freezegun** — Time mocking
- **mypy**, **ruff**, **isort** — Code quality

### Frontend
- **React** 18+, **React Router** — UI framework
- **TypeScript** — Type safety
- **Vite** — Build tool
- **Tailwind CSS** — Styling
- **shadcn/ui** (inferred) — Component library

---

## Build & Run

### Python Backend

#### Install
```bash
# From source
pip install -e .

# Or from PyPI
pip install skyvern
```

#### Run
```bash
# Quickstart (UI + server)
skyvern quickstart

# Just the server
python -m skyvern.forge.api_app  # or uvicorn skyvern.forge.api_app:app

# Run a task via CLI
skyvern run-task --url "https://example.com" --goal "Do something"

# Run a workflow
skyvern run-workflow --workflow-id "wf_123"
```

#### Database Setup
```bash
# Run migrations
alembic upgrade head

# Check migration status
./run_alembic_check.sh
```

#### Docker
```bash
# Backend
docker build -t skyvern:latest .

# Frontend
docker build -f Dockerfile.ui -t skyvern-ui:latest .

# Compose (both services)
docker-compose up
```

### Frontend

#### Install
```bash
cd skyvern-frontend
npm install
```

#### Dev
```bash
npm run dev
```

#### Build
```bash
npm run build
```

---

## Patterns & Conventions

### Code Organization

1. **Separation of Concerns**
   - `forge/agent.py` — Business logic, orchestration
   - `services/` — Concrete implementations of actions
   - `forge/sdk/routes/` — HTTP request handling
   - `forge/sdk/db/` — Data persistence

2. **Async/Await Throughout**
   - All I/O operations are async (database, HTTP, browser control)
   - Event loop managed by Uvicorn (production) or asyncio (dev/testing)
   - Database operations use asyncpg + SQLAlchemy async

3. **Lazy Imports in Public API** (`__init__.py`)
   - Symbols exported at package level use `_lazy_imports` dict
   - Reduces startup time and import cycles
   - Example: `from skyvern import Skyvern, workflow, action`

### Error Handling

1. **Custom Exceptions** (`exceptions.py`)
   - `SkyvernHTTPException` — HTTP-friendly exceptions with status codes
   - Caught by middleware and returned as JSON responses
   - Format: `{"message": "...", "status": 400}`

2. **Validation**
   - Pydantic validates all request schemas automatically
   - Custom validators on schema classes (e.g., `TaskRequest.validate_url()`)

3. **Logging**
   - Structured logging with `structlog`
   - Log context includes request ID, organization ID, user info
   - Levels: DEBUG, INFO, WARNING, ERROR, CRITICAL

### Database Access Patterns

1. **Agent DB** (`forge/sdk/db/agent_db.py`)
   - CRUD operations for agent domain objects
   - Async session management
   - Methods: `create_task()`, `get_step()`, `update_task_status()`, etc.

2. **SQLAlchemy Models**
   - `Base` declarative base with async attributes (`AsyncAttrs`)
   - Relationships defined via ForeignKey and relationships
   - Indexes on frequently queried columns (org_id, created_at, etc.)

3. **Session Lifecycle**
   - Sessions created per request via dependency injection
   - Automatically closed by FastAPI's dependency system

### Testing

1. **Unit Tests** (`tests/unit/`)
   - Mock database, LLM calls, browser operations
   - Use `pytest` with async support (`pytest-asyncio`)
   - Fixtures in `conftest.py`

2. **SDK Tests** (`tests/sdk/python_sdk/`)
   - Test high-level library API (Skyvern class, browser page, actions)
   - May use real browser or mocked depending on test

3. **Smoke Tests** (`tests/smoke_tests/`)
   - End-to-end integration tests
   - Verify full task execution pipeline

### Naming Conventions

- **Classes**: PascalCase (Agent, TaskModel, SkyvernBrowser)
- **Functions/variables**: snake_case (create_task, browser_session_id)
- **Constants**: UPPER_SNAKE_CASE (MAX_STEPS_PER_RUN, ScrapeType)
- **Enums**: PascalCase (TaskStatus, StepStatus)
- **Private**: Leading underscore (_private_method, _hidden_field)

### Type Hints

- Type hints used throughout (PEP 484)
- Generic types: `dict[str, Any]`, `list[T]`, `Optional[T]` (or `T | None`)
- Protocol types for interfaces (e.g., `ArtifactStorage` protocol)
- **mypy** for static type checking (config in `mypy.ini`)

---

## Open Questions

1. **Streaming API Details**: The `forge/sdk/routes/streaming/` directory has multiple channel types (CDP, execution, exfiltration, VNC, screenshot). How do these differ and when is each used? Need to explore `streaming/channels/` files.

2. **Workflow Block Execution**: How does the `WorkflowService` handle nested blocks (e.g., LoopBlock containing ActionBlocks)? Is there a recursive traversal or a linearized queue?

3. **Task V2 Architecture**: The codebase has both `task_v1_service.py` and `task_v2_service.py`. What are the architectural differences? When is each used?

4. **Browser Session Persistence**: How are persistent browser sessions maintained across task executions? Is there a separate manager service?

5. **LLM Provider Selection**: How does `LLMConfigRegistry` work? Can users easily swap between OpenAI, Claude, and Vertex AI?

6. **Credential Encryption**: How are stored credentials (passwords, API keys) encrypted? What's the encryption strategy?

7. **Frontend State Management**: Does the React frontend use Redux, Zustand, or Context API for state management? Need to check `src/hooks/` and component files.

8. **CI/CD Pipeline**: What's the deployment pipeline? The `.github/workflows/` directory has multiple CI files — are they for cloud vs. local deployment?

9. **Observability**: The `forge/api_app.py` mentions OpenTelemetry setup — is this fully integrated? What metrics/traces are exported?

10. **Multi-tenancy**: The database models include `organization_id` fields. How is multi-tenancy enforced at the API level? Is there middleware that validates org context?

---

## Recent Development

Based on git history:

- **Latest commit** (9721f0e): Manicule initialization (session.json update)
- **Previous** (b33bc03): "Docs" commit — moved context from `skyvern-code` to `skyvern` directory, added README
- **Earlier**: Multiple reset/reinitialization commits, suggesting this is a documentation generation project that periodically imports Skyvern codebase snapshots

**Key takeaway**: This is a documentation generation repository that extracts and indexes Skyvern source code. Not the canonical Skyvern repository.

