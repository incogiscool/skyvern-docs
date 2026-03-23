# Context Index

## Overview

Skyvern is a browser automation platform that uses LLMs and computer vision to interact with websites. It replaces brittle XPath/DOM-parsing scripts with vision-based AI agents that can navigate any website without site-specific code. The core stack is: **Python 3.11+** backend (FastAPI, SQLAlchemy/PostgreSQL, Playwright), **React/TypeScript** frontend, and a multi-LLM architecture using LiteLLM routing (OpenAI, Anthropic, Azure, Gemini, Bedrock, etc.). The system ships as three products: Skyvern Cloud (`app.skyvern.com`), a self-hosted Docker deployment, and a Python SDK (`pip install skyvern`).

---

## File Tree

```
/source
├── skyvern/                      # Main Python package
│   ├── __main__.py               # CLI entrypoint: calls skyvern._cli_bootstrap → cli_app
│   ├── _version.py               # Package version (1.0.24)
│   ├── config.py                 # Settings class (Pydantic BaseSettings) — all env vars
│   ├── constants.py              # Global constants (paths, ScrapeType enum)
│   ├── exceptions.py             # Global exceptions (SkyvernHTTPException etc.)
│   ├── analytics.py              # PostHog analytics wrapper
│   ├── cli/                      # CLI commands (Typer)
│   │   ├── commands/__init__.py  # Main cli_app with all sub-commands registered
│   │   ├── run_commands.py       # `skyvern run` group
│   │   ├── quickstart.py         # `skyvern quickstart`
│   │   ├── setup_commands.py     # `skyvern setup` (MCP registration)
│   │   ├── mcp_commands.py       # `skyvern mcp`
│   │   ├── workflow.py           # `skyvern workflow`
│   │   ├── tasks.py              # `skyvern tasks`
│   │   ├── credentials.py        # `skyvern credentials` (add/list/delete)
│   │   ├── skill_commands.py     # `skyvern skill`
│   │   ├── mcp_tools/            # MCP server tools implementation
│   │   │   ├── browser.py        # Browser action tools
│   │   │   ├── session.py        # Session management tools
│   │   │   ├── workflow.py       # Workflow tools
│   │   │   ├── credential.py     # Credential tools
│   │   │   └── folder.py         # Folder tools
│   │   └── core/                 # Shared CLI core utilities
│   ├── client/                   # Auto-generated Fern Python SDK client
│   │   ├── client.py             # Main Skyvern / AsyncSkyvern client classes
│   │   ├── raw_client.py         # Low-level HTTP client
│   │   ├── environment.py        # SkyvernEnvironment enum (CLOUD/STAGING/LOCAL)
│   │   └── types/                # Generated Pydantic types (hundreds of files)
│   ├── forge/                    # Core server: FastAPI app, agents, services
│   │   ├── api_app.py            # FastAPI app factory (create_api_app)
│   │   ├── forge_app.py          # ForgeApp container (all shared services)
│   │   ├── forge_app_initializer.py  # start_forge_app() initializer
│   │   ├── agent.py              # ForgeAgent — core web navigation agent
│   │   ├── agent_functions.py    # AgentFunction — task/step execution hooks
│   │   ├── prompts.py            # Jinja2 prompt engine
│   │   ├── async_operations.py   # Async operation pool for parallel steps
│   │   └── sdk/
│   │       ├── routes/           # FastAPI routers
│   │       │   ├── routers.py    # Defines base_router, legacy_base_router, legacy_v2_router
│   │       │   ├── agent_protocol.py  # Main API endpoints (tasks, workflows, runs, etc.)
│   │       │   ├── credentials.py     # Credential CRUD + TOTP endpoints
│   │       │   ├── browser_sessions.py # Browser session endpoints
│   │       │   ├── browser_profiles.py # Browser profile endpoints
│   │       │   ├── debug_sessions.py  # Debug session endpoints
│   │       │   ├── scripts.py         # Script management endpoints
│   │       │   ├── sdk.py             # SDK action endpoint
│   │       │   ├── webhooks.py        # Webhook handling
│   │       │   └── streaming/         # WebSocket/SSE streaming channels
│   │       ├── schemas/           # Pydantic request/response models
│   │       │   ├── tasks.py       # TaskBase, TaskRequest, Task, TaskResponse, TaskStatus
│   │       │   ├── runs.py        # Run model (task_run_id, run_id, RunType)
│   │       │   ├── credentials.py # CredentialType, PasswordCredential, CreateCredentialRequest
│   │       │   ├── browser_profiles.py  # BrowserProfile, CreateBrowserProfileRequest
│   │       │   ├── persistent_browser_sessions.py # PersistentBrowserSession
│   │       │   ├── workflow_runs.py     # WorkflowRun, WorkflowRunStatus
│   │       │   ├── debug_sessions.py    # DebugSession
│   │       │   └── scripts.py          # Script schemas
│   │       ├── db/                # Database layer
│   │       │   ├── models.py      # All SQLAlchemy ORM models (40+ tables)
│   │       │   ├── agent_db.py    # AgentDB — data access layer (large file)
│   │       │   ├── enums.py       # OrganizationAuthTokenType, TaskType, WorkflowRunTriggerType
│   │       │   ├── id.py          # ID generators (generate_task_id, etc.)
│   │       │   └── base_alchemy_db.py  # Base async SQLAlchemy setup
│   │       ├── api/               # External API clients
│   │       │   ├── llm/           # LLM abstraction layer
│   │       │   │   ├── config_registry.py  # LLMConfigRegistry — registers all model configs
│   │       │   │   ├── api_handler.py      # LLMAPIHandler — calls LiteLLM
│   │       │   │   ├── api_handler_factory.py  # LLMAPIHandlerFactory
│   │       │   │   └── models.py           # LLMConfig, LLMRouterConfig
│   │       │   ├── aws.py         # AWS client
│   │       │   ├── azure.py       # Azure client factory
│   │       │   └── custom_credential_client.py  # Custom credential HTTP client
│   │       ├── services/          # Business logic services
│   │       │   ├── org_auth_service.py  # Authentication (API key + JWT)
│   │       │   ├── credentials.py       # Credential management service
│   │       │   └── credential/          # Credential vault backends
│   │       │       ├── bitwarden_credential_service.py
│   │       │       ├── azure_credential_vault_service.py
│   │       │       └── custom_credential_vault_service.py
│   │       ├── workflow/          # Workflow engine
│   │       │   ├── service.py     # WorkflowService — create/run/manage workflows
│   │       │   ├── context_manager.py   # WorkflowContextManager
│   │       │   └── models/
│   │       │       ├── block.py   # All Block implementations (4000+ lines)
│   │       │       ├── parameter.py  # Parameter types
│   │       │       └── workflow.py   # Workflow, WorkflowDefinition, WorkflowRun
│   │       ├── artifact/          # Artifact storage
│   │       │   ├── manager.py     # ArtifactManager
│   │       │   ├── models.py      # ArtifactType enum
│   │       │   └── storage/       # Storage backends (local, S3, Azure Blob)
│   │       └── core/
│   │           ├── security.py    # JWT creation, webhook signature generation
│   │           ├── skyvern_context.py  # Request context (org_id, task_id)
│   │           └── permissions/   # Permission checker framework
│   ├── webeye/                   # Browser automation layer
│   │   ├── browser_manager.py    # BrowserManager base class
│   │   ├── browser_factory.py    # Browser instance creation
│   │   ├── browser_state.py      # BrowserState, BrowserContext
│   │   ├── persistent_sessions_manager.py  # PersistentSessionsManager base
│   │   ├── default_persistent_sessions_manager.py  # Default implementation
│   │   ├── real_browser_manager.py  # RealBrowserManager (local browser)
│   │   ├── scraper/
│   │   │   └── scraper.py        # scrape_website(), element tree building
│   │   └── actions/
│   │       ├── actions.py        # Action classes (ClickAction, InputTextAction, etc.)
│   │       ├── handler.py        # Action executor
│   │       ├── handler_utils.py  # Action handler utilities
│   │       ├── parse_actions.py  # LLM response → Action parsing
│   │       ├── responses.py      # ActionResult types
│   │       └── caching.py        # Action caching
│   ├── services/                 # Top-level services
│   │   ├── run_service.py (or similar)  # Run orchestration
│   │   └── browser_recording/   # Browser session recording
│   ├── schemas/                  # Public API schemas (used in OpenAPI)
│   │   ├── runs.py               # TaskRunRequest, WorkflowRunRequest, RunStatus, RunEngine
│   │   ├── workflows.py          # BlockType enum, WorkflowDefinition schema
│   │   ├── artifacts.py          # Artifact response schemas
│   │   ├── steps.py              # Step schemas
│   │   ├── folders.py            # Folder schemas
│   │   └── run_blocks.py         # Block run schemas
│   └── errors/                   # Error types
├── skyvern-frontend/             # React 18 + TypeScript + Vite UI
│   └── src/
│       ├── router.tsx            # React Router config (all page routes)
│       ├── routes/               # Page components
│       │   ├── tasks/            # Task list, create, detail pages
│       │   ├── workflows/        # Workflow list, editor, run detail, debugger
│       │   ├── runs/             # Unified run router
│       │   ├── credentials/      # Credentials page
│       │   ├── browserSessions/  # Browser session management
│       │   ├── history/          # Run history page
│       │   ├── discover/         # Workflow templates/discover page
│       │   ├── schedules/        # Scheduled workflow management
│       │   └── settings/         # User settings
│       ├── api/                  # API client (axios-based)
│       ├── components/           # Shared UI components
│       ├── hooks/                # Custom React hooks
│       ├── store/                # Zustand/context state management
│       └── types.ts              # Shared TypeScript types
├── skyvern-ts/
│   └── client/                   # Auto-generated TypeScript SDK (@skyvern/client)
├── fern/                         # API/documentation spec (Fern framework)
│   ├── openapi/
│   │   └── skyvern_openapi.json  # OpenAPI 3.x spec (auto-updated from API)
│   ├── docs.yml                  # Fern docs navigation config
│   ├── generators.yml            # SDK generator config (Python + TypeScript)
│   ├── introduction.mdx          # Root intro page
│   ├── getting-started/          # Quickstart, Skyvern in Action, Prompting Guide
│   ├── running-tasks/            # Task docs (run-tasks, visualizing, cancel, webhooks, etc.)
│   ├── workflows/                # Workflow docs (blocks, parameters, manage, run)
│   ├── credentials/              # Credential docs (passwords, credit cards, TOTP, Bitwarden)
│   ├── browser-sessions/         # Browser session docs
│   ├── observability/            # Observability overview
│   └── integrations/             # MCP, CLI, Zapier, Make, N8N, Workato, Ollama
├── alembic/                      # Database migrations (PostgreSQL)
│   └── versions/                 # 100+ migration files (2024-01 to present)
├── tests/
│   ├── unit/                     # Pytest unit tests
│   ├── unit_tests/               # More unit tests (100+ test files)
│   ├── sdk/                      # SDK integration tests
│   └── smoke_tests/              # End-to-end smoke tests
├── docker-compose.yml            # Docker Compose setup (postgres + skyvern + skyvernui)
├── Dockerfile                    # Main Skyvern server image
├── Dockerfile.ui                 # Frontend image
├── pyproject.toml                # Python project config, dependencies, scripts
├── alembic.ini                   # Alembic config
├── .env.example                  # Environment variable documentation
├── CLAUDE.md                     # AI coding assistant guidance
└── AGENTS.md                     # Agent instructions
```

---

## Architecture

### System Design

```
External Client (SDK/REST)
        │
        ▼
FastAPI Server (port 8000)
  ├── /v1/*  (base_router)          ← Current public API
  ├── /api/v1/* (legacy_base_router) ← Legacy endpoints
  └── /api/v2/* (legacy_v2_router)   ← Legacy v2
        │
        ▼
ForgeApp (singleton container for all services)
  ├── DATABASE (AgentDB → PostgreSQL via asyncpg/psycopg)
  ├── LLM_API_HANDLER (LiteLLM routing to OpenAI/Anthropic/Gemini/etc.)
  ├── BROWSER_MANAGER (Playwright browser lifecycle)
  ├── ARTIFACT_MANAGER (screenshots, recordings, files)
  ├── WORKFLOW_SERVICE (workflow execution engine)
  ├── WORKFLOW_CONTEXT_MANAGER (per-run context)
  └── PERSISTENT_SESSIONS_MANAGER (long-lived browser sessions)
        │
        ▼
ForgeAgent
  ├── create_task_and_step_from_block()
  ├── execute_step()  → screenshot → scrape → LLM → actions → record
  └── agent_step()    → multi-phase step loop
        │
        ▼
Browser Engine (webeye)
  ├── scraper.py: scrape_website() → element tree
  ├── actions/handler.py: execute ClickAction, InputTextAction, etc.
  └── Playwright Page API
```

### Key Abstractions

- **Task (v1)**: Single-URL, single-goal automation. Legacy API: `/api/v1/tasks`. New API: `POST /v1/run/tasks`.
- **Task (v2)**: Same as v1 but uses the improved Skyvern 2.0 agent pipeline. Both represented by `tsk_v2_*` IDs.
- **Workflow**: Multi-block automation (YAML/JSON definition). Has a `workflow_permanent_id` (wpid_*) across versions.
- **WorkflowRun**: An execution instance of a Workflow. ID prefix: `wr_*`.
- **Block**: An individual step in a workflow (TaskBlock, ForLoopBlock, ConditionalBlock, CodeBlock, etc.).
- **BrowserSession**: A persistent browser instance that can be reused across multiple runs. ID prefix: `pbs_*`.
- **BrowserProfile**: A saved browser state (cookies, localStorage) that can be applied to future runs.
- **Credential**: Securely stored authentication data (password, credit card, secret).
- **Step**: A single LLM call + action execution cycle within a task. ID prefix: `stp_*`.
- **Artifact**: Any file produced during execution (screenshot, recording, HTML, LLM prompts/responses).

### Data Flow

1. Client sends `POST /v1/run/tasks` with `TaskRunRequest`
2. API layer validates, creates `TaskModel` + `StepModel` in DB
3. `WorkflowService.execute_workflow()` or `ForgeAgent.execute_step()` is called
4. Agent takes screenshot → scrapes element tree → sends to LLM → parses actions
5. `ActionHandler` executes actions via Playwright
6. Results stored as artifacts in S3/Azure/local storage
7. Webhook fired to `webhook_url` on completion

---

## Key Concepts

### Task Execution Engines (RunEngine)
- **`skyvern-2.0`** (default) — Vision LLM agent; state-of-art on WebVoyager benchmark
- **`skyvern-1.0`** — Simpler agent, good for single-goal tasks
- **`openai-cua`** — OpenAI CUA model
- **`anthropic-cua`** — Anthropic Claude 3.7 with computer use tool
- **`ui-tars`** — UI-TARS / Seed1.5-VL model via Doubao API
- Defined in: `skyvern/schemas/runs.py:L341`

### Block Types (Workflow Building Blocks)
Defined in `skyvern/schemas/workflows.py:L358`:
- `task` — Web navigation/extraction (TaskBlock)
- `task_v2` — V2 agent task
- `for_loop` — Iterate over a list
- `conditional` — Branch on a condition
- `code` — Execute arbitrary Python code
- `text_prompt` — Call LLM with a text prompt
- `download_to_s3` — Download file to S3
- `upload_to_s3` — Upload file to S3
- `file_upload` — Upload local file
- `send_email` — Send email
- `file_url_parser` — Parse a file at a URL
- `pdf_parser` — Extract text from PDF
- `validation` — Validate task completion
- `action` — Direct browser action
- `navigation` — Browser navigation
- `extraction` — Data extraction from page
- `login` — Login with credentials
- `wait` — Wait N seconds
- `file_download` — Download a file
- `goto_url` — Navigate to URL
- `http_request` — Make an HTTP request
- `human_interaction` — Pause for human
- `print_page` — Print/capture page
- `workflow_trigger` — Trigger another workflow
Full implementations in: `skyvern/forge/sdk/workflow/models/block.py`

### Parameters (Workflow Inputs)
Defined in `skyvern/forge/sdk/workflow/models/parameter.py`:
- `WorkflowParameter` — User-supplied input at run time
- `OutputParameter` — Output from a previous block
- `ContextParameter` — Context values (workflow_run_id, etc.)
- `AWSSecretParameter` — Value fetched from AWS Secrets Manager
- `BitwardenLoginCredentialParameter` — From Bitwarden
- `BitwardenSensitiveInformationParameter` — Bitwarden secure note
- `BitwardenCreditCardDataParameter` — Bitwarden credit card
- `CredentialParameter` — Skyvern credential vault reference
- `OnePasswordCredentialParameter` — 1Password item
- `AzureVaultCredentialParameter` — Azure Key Vault

### Credentials
Defined in `skyvern/forge/sdk/schemas/credentials.py`:
- **CredentialType**: `password`, `credit_card`, `secret`
- **CredentialVaultType**: `bitwarden`, `azure_vault`, `custom`
- **TotpType**: `authenticator`, `email`, `text`, `none`
- Security invariant: API responses **never** return raw credentials — only non-sensitive metadata (username, last_four, secret_label)

### LLM Configuration
- `LLMConfigRegistry` (`skyvern/forge/sdk/api/llm/config_registry.py`) — singleton registry
- Configs registered at startup based on enabled env vars (`ENABLE_OPENAI`, `ENABLE_ANTHROPIC`, etc.)
- `LLM_KEY` env var selects the primary model
- `SECONDARY_LLM_KEY` for lightweight operations (selection, SVG conversion)
- Uses LiteLLM for unified API across providers
- Supports router configs for load balancing/fallback

### Browser Sessions
Defined in `skyvern/forge/sdk/schemas/persistent_browser_sessions.py`:
- **Status**: `created`, `running`, `failed`, `completed`, `timeout`, `retry`
- **PersistentBrowserType**: `msedge`, `chrome`, `stealth-chromium`
- **Extensions**: `ad-blocker`, `captcha-solver`
- Sessions can be reused across tasks/workflow runs via `browser_session_id`

### Proxy Locations
Defined in `skyvern/schemas/runs.py:L36`:
- `RESIDENTIAL` (US), `RESIDENTIAL_GB`, `RESIDENTIAL_DE`, `RESIDENTIAL_FR`, `RESIDENTIAL_IN`, `RESIDENTIAL_JP`, etc.
- Also `US-CA`, `US-NY`, `US-TX`, `US-FL`, `US-WA` (state-level)
- `GeoTarget` object for granular city/state (`{"country": "US", "subdivision": "CA", "city": "San Francisco"}`)
- `NONE` to disable proxy

---

## API Surface

All routes are under `/v1/` (current) or `/api/v1/` (legacy, include_in_schema=False).

### Agent / Tasks (base_router, `/v1/`)

| Method | Path | Function | Description |
|--------|------|----------|-------------|
| POST | `/v1/run/tasks` | `run_task` | Run a task (main endpoint) |
| POST | `/v1/run/workflows` | `run_workflow` | Run a workflow |
| GET | `/v1/runs/{run_id}` | `get_run` | Get a task or workflow run |
| POST | `/v1/runs/{run_id}/cancel` | `cancel_run` | Cancel a run |
| POST | `/v1/runs/{run_id}/retry_webhook` | `retry_run_webhook` | Retry webhook delivery |
| GET | `/v1/runs/{run_id}/artifacts` | `get_run_artifacts` | List artifacts for a run |
| GET | `/v1/runs/{run_id}/timeline` | `get_run_timeline` | Get run timeline |
| GET | `/v1/artifacts/{artifact_id}` | `get_artifact` | Get an artifact |
| GET | `/v1/artifacts/{artifact_id}/content` | `get_artifact_content` | Get artifact content |

Defined in: `skyvern/forge/sdk/routes/agent_protocol.py:L143-L560`

### Workflows

| Method | Path | Function | Description |
|--------|------|----------|-------------|
| POST | `/v1/workflows` | `create_workflow` | Create a workflow |
| POST | `/v1/workflows/from_prompt` | `create_workflow_from_prompt` | Create from natural language |
| POST | `/v1/workflows/{workflow_id}` | `update_workflow` | Update a workflow |
| POST | `/v1/workflows/{workflow_id}/delete` | `delete_workflow` | Delete a workflow |
| GET | `/v1/workflows` | `get_workflows` | List workflows |
| GET | `/v1/workflows/{workflow_permanent_id}` | `get_workflow` | Get workflow |
| GET | `/v1/workflows/{workflow_permanent_id}/versions` | `get_workflow_versions` | Get all versions |
| GET | `/v1/workflows/runs` | `get_workflow_runs` | List workflow runs |
| POST | `/v1/workflows/runs/{workflow_run_id}/continue` | `continue_workflow_run` | Continue a paused run |

Defined in: `skyvern/forge/sdk/routes/agent_protocol.py:L570-L1069` and `L2336-L2900`

### Credentials

| Method | Path | Function | Description |
|--------|------|----------|-------------|
| POST | `/v1/credentials` | `create_credential` | Create a credential |
| GET | `/v1/credentials` | `get_credentials` | List credentials |
| GET | `/v1/credentials/{credential_id}` | `get_credential` | Get credential metadata |
| PATCH | `/v1/credentials/{credential_id}` | `rename_credential` | Rename a credential |
| POST | `/v1/credentials/{credential_id}/delete` | `delete_credential` | Delete a credential |
| POST | `/v1/credentials/totp` | `send_totp_code` | Send/submit a TOTP code |
| GET | `/v1/credentials/totp` | `get_totp_codes` | List TOTP codes |
| POST | `/v1/credentials/test_login` | `test_login` | Test a credential |
| POST | `/v1/credentials/test` | `test_credential` | Test a credential (v2) |

Defined in: `skyvern/forge/sdk/routes/credentials.py:L122-L960`

### Browser Sessions

| Method | Path | Function | Description |
|--------|------|----------|-------------|
| POST | `/v1/browser_sessions` | `create_browser_session` | Create a browser session |
| GET | `/v1/browser_sessions` | `get_browser_sessions` | List browser sessions |
| GET | `/v1/browser_sessions/{browser_session_id}` | `get_browser_session` | Get a session |
| POST | `/v1/browser_sessions/{browser_session_id}/close` | `close_browser_session` | Close a session |

Defined in: `skyvern/forge/sdk/routes/browser_sessions.py:L29-L245`

### Browser Profiles

| Method | Path | Function | Description |
|--------|------|----------|-------------|
| POST | `/v1/browser_profiles` | `create_browser_profile` | Create a profile |
| GET | `/v1/browser_profiles` | `list_browser_profiles` | List profiles |
| GET | `/v1/browser_profiles/{profile_id}` | `get_browser_profile` | Get a profile |
| DELETE | `/v1/browser_profiles/{profile_id}` | `delete_browser_profile` | Delete a profile |

Defined in: `skyvern/forge/sdk/routes/browser_profiles.py:L49-L355`

### Folders

| Method | Path | Function | Description |
|--------|------|----------|-------------|
| POST | `/v1/folders` | `create_folder` | Create a folder |
| GET | `/v1/folders` | `get_folders` | List folders |
| GET | `/v1/folders/{folder_id}` | `get_folder` | Get a folder |
| PUT | `/v1/folders/{folder_id}` | `update_folder` | Update a folder |
| DELETE | `/v1/folders/{folder_id}` | `delete_folder` | Delete a folder |
| PUT | `/v1/workflows/{workflow_permanent_id}/folder` | `update_workflow_folder` | Move workflow to folder |

Defined in: `skyvern/forge/sdk/routes/agent_protocol.py:L1079-L1340`

### Scripts (hidden in public docs)

| Method | Path | Function | Description |
|--------|------|----------|-------------|
| POST | `/v1/scripts` | `create_script` | Create a script |
| GET | `/v1/scripts` | `get_scripts` | List scripts |
| GET | `/v1/scripts/{script_id}` | `get_script` | Get a script |
| POST | `/v1/scripts/{script_id}/run` | `run_script` | Run a script |
| POST | `/v1/scripts/{script_id}/deploy` | `deploy_script` | Deploy a script |

Defined in: `skyvern/forge/sdk/routes/scripts.py:L206-L877`

### Files

| Method | Path | Function | Description |
|--------|------|----------|-------------|
| POST | `/v1/upload_file` | - | Upload a file |

### Utilities

| Method | Path | Function | Description |
|--------|------|----------|-------------|
| GET | `/v1/heartbeat` | `heartbeat` | Health check |
| GET | `/v1/version` | `get_version` | Get server version |
| GET | `/v1/models` | `models` | List available models |
| POST | `/v1/utilities/curl-to-http` | `convert_curl_to_http` | Convert cURL to HTTP block |

---

## Data Model

All SQLAlchemy models defined in `skyvern/forge/sdk/db/models.py`.

### Core Tables

| Model | Table | Key Fields |
|-------|-------|-----------|
| `TaskModel` | `tasks` | `task_id`, `organization_id`, `status`, `url`, `navigation_goal`, `data_extraction_goal`, `extracted_information`, `workflow_run_id`, `browser_session_id`, `model` |
| `StepModel` | `steps` | `step_id`, `task_id`, `organization_id`, `status`, `order`, `retry_index`, `input_token_count`, `output_token_count`, `step_cost` |
| `OrganizationModel` | `organizations` | `organization_id`, `organization_name`, `webhook_callback_url`, `max_steps_per_run` |
| `OrganizationAuthTokenModel` | `organization_auth_tokens` | `id`, `organization_id`, `token_type`, `token`, `encrypted_token` |
| `ArtifactModel` | `artifacts` | `artifact_id`, `task_id`, `step_id`, `workflow_run_id`, `artifact_type`, `uri`, `run_id` |
| `WorkflowModel` | `workflows` | `workflow_id`, `workflow_permanent_id`, `version`, `title`, `workflow_definition`, `status`, `generate_script`, `run_with`, `ai_fallback` |
| `WorkflowRunModel` | `workflow_runs` | `workflow_run_id`, `workflow_id`, `workflow_permanent_id`, `status`, `failure_reason`, `proxy_location`, `browser_session_id`, `parent_workflow_run_id` |
| `WorkflowRunBlockModel` | `workflow_run_blocks` | `workflow_run_block_id`, `workflow_run_id`, `block_type`, `label`, `status`, `output_parameter_value` |
| `PersistentBrowserSessionModel` | `persistent_browser_sessions` | `persistent_browser_session_id`, `organization_id`, `status`, `browser_address`, `timeout_minutes`, `proxy_location` |
| `BrowserProfileModel` | `browser_profiles` | `browser_profile_id`, `organization_id`, `name`, `description` |
| `CredentialModel` | `credentials` | `credential_id`, `organization_id`, `credential_type`, `credential_vault_type` |
| `TaskV2Model` | `task_v2` | `task_v2_id`, `organization_id`, `status`, `url`, `prompt`, `generate_script` |
| `ThoughtModel` | `thoughts` | `thought_id`, `observer_cruise_id`, `thought_type`, `input_token_count`, `output_token_count` |
| `DebugSessionModel` | `debug_sessions` | `debug_session_id`, `organization_id`, `workflow_permanent_id`, `status` |
| `ScriptModel` | `scripts` | `script_id`, `organization_id`, `title`, `status` |
| `TaskRunModel` | `task_runs` | `task_run_id`, `organization_id`, `run_id`, `task_run_type` |
| `FolderModel` | `folders` | `folder_id`, `organization_id`, `title` |
| `ActionModel` | `actions` | `action_id`, `task_id`, `step_id`, `action_type`, `confidence_float`, `organization_id` |

### Workflow Parameter Tables

| Model | Table | Purpose |
|-------|-------|---------|
| `WorkflowParameterModel` | `workflow_parameters` | User-defined workflow inputs |
| `OutputParameterModel` | `output_parameters` | Block output references |
| `AWSSecretParameterModel` | `aws_secret_parameters` | AWS Secrets Manager refs |
| `BitwardenLoginCredentialParameterModel` | `bitwarden_login_credential_parameters` | Bitwarden login creds |
| `BitwardenSensitiveInformationParameterModel` | `bitwarden_sensitive_information_parameters` | Bitwarden secure notes |
| `CredentialParameterModel` | `credential_parameters` | Skyvern vault refs |
| `OnePasswordCredentialParameterModel` | `onepassword_credential_parameters` | 1Password refs |
| `AzureVaultCredentialParameterModel` | `azure_vault_credential_parameters` | Azure Key Vault refs |

### ID Prefixes
Defined in `skyvern/forge/sdk/db/id.py`:
- `tsk_` — Task (v1)
- `tsk_v2_` — Task (v2)
- `stp_` — Step
- `wr_` — Workflow run
- `wpid_` — Workflow permanent ID
- `wf_` — Workflow version
- `pbs_` — Persistent browser session
- `bp_` — Browser profile
- `crd_` — Credential
- `art_` — Artifact
- `org_` — Organization
- `dbg_` — Debug session

---

## Auth & Middleware

### Authentication
- **Primary**: `x-api-key` header with JWT token
- **Secondary**: `Authorization: Bearer <token>` header (for cloud/UI users)
- Auth is enforced via FastAPI `Depends(org_auth_service.get_current_org)` on every protected endpoint
- Defined in: `skyvern/forge/sdk/services/org_auth_service.py:L34`
- `get_current_org()` → decodes JWT → looks up `OrganizationAuthTokenModel` → returns `Organization`
- Tokens cached in TTL cache (1 hour, 128 entries) for performance
- In `ENV=local`, internal auth routes are also enabled for dev convenience

### API Key Structure
- JWT-encoded, signed with `SECRET_KEY` using `SIGNATURE_ALGORITHM` (HS256)
- Payload contains organization_id
- Validated against DB (`organization_auth_tokens` table)

### Webhook Signatures
- Generated via HMAC-SHA256 using API key
- Headers: `x-skyvern-timestamp`, `x-skyvern-signature`
- Defined in: `skyvern/forge/sdk/core/security.py:L43`

### Middleware Stack (FastAPI)
1. `CORSMiddleware` — Configurable via `ALLOWED_ORIGINS` env var
2. `RawContextMiddleware` (starlette-context) — Injects `ExecutionDatePlugin`
3. `log_raw_request_middleware` — Request logging (when `LOG_RAW_API_REQUESTS=true`)
4. Exception handlers: `NotFoundError`, `SkyvernHTTPException`, `ValidationError`, general `Exception`

---

## Configuration

All settings in `skyvern/config.py` as a `Settings(BaseSettings)` class, loaded from `.env` file.

| Variable | Purpose | Default |
|----------|---------|---------|
| `ENV` | Environment name | `local` |
| `LLM_KEY` | Primary LLM model key | (required) |
| `SECONDARY_LLM_KEY` | Secondary/lighter LLM | (same as LLM_KEY) |
| `ENABLE_OPENAI` | Enable OpenAI provider | `false` |
| `OPENAI_API_KEY` | OpenAI API key | — |
| `ENABLE_ANTHROPIC` | Enable Anthropic provider | `false` |
| `ANTHROPIC_API_KEY` | Anthropic API key | — |
| `ENABLE_AZURE` | Enable Azure OpenAI | `false` |
| `AZURE_DEPLOYMENT` | Azure deployment name | — |
| `AZURE_API_KEY` | Azure API key | — |
| `ENABLE_GEMINI` | Enable Google Gemini | `false` |
| `GEMINI_API_KEY` | Gemini API key | — |
| `DATABASE_STRING` | PostgreSQL connection URL | `postgresql+psycopg://skyvern@localhost/skyvern` |
| `DATABASE_REPLICA_STRING` | Read replica URL | `null` |
| `PORT` | API server port | `8000` |
| `SECRET_KEY` | JWT signing key | `PLACEHOLDER` |
| `ALLOWED_ORIGINS` | CORS origins | `["*"]` |
| `SKYVERN_STORAGE_TYPE` | Artifact storage backend | `local` |
| `ARTIFACT_STORAGE_PATH` | Local artifact path | `./skyvern/artifacts` |
| `AWS_S3_BUCKET_ARTIFACTS` | S3 bucket for artifacts | `skyvern-artifacts` |
| `AZURE_STORAGE_ACCOUNT_NAME` | Azure Storage account | `null` |
| `MAX_STEPS_PER_RUN` | Max steps per task | `10` |
| `MAX_STEPS_PER_TASK_V2` | Max steps for v2 tasks | `25` |
| `BROWSER_TYPE` | Browser type | `chromium-headful` |
| `BROWSER_ACTION_TIMEOUT_MS` | Action timeout | `5000` |
| `BROWSER_WIDTH` / `BROWSER_HEIGHT` | Viewport size | `1920×1080` |
| `BROWSER_LOCALE` | Browser locale | `null` |
| `BROWSER_TIMEZONE` | Browser timezone | `America/New_York` |
| `ENABLE_PROXY` | Enable proxy routing | `false` |
| `HOSTED_PROXY_POOL` | Proxy pool URLs | `""` |
| `REDIS_URL` | Redis connection URL | `redis://localhost:6379/0` |
| `SKYVERN_TELEMETRY` | Enable PostHog analytics | `true` |
| `JSON_LOGGING` | JSON structured logging | `false` |
| `LOG_LEVEL` | Log level | `INFO` |
| `ENCRYPTOR_AES_SECRET_KEY` | AES encryption key | `fillmein` |
| `DEBUG_SESSION_TIMEOUT_MINUTES` | Debug session timeout | `20` |
| `OTEL_ENABLED` | Enable OpenTelemetry | `false` |
| `ENABLE_CLEANUP_CRON` | Enable stale data cleanup | `false` |

LLM keys supported (registered in `config_registry.py`):
- OpenAI: `OPENAI_GPT5`, `OPENAI_GPT5_MINI`, `OPENAI_GPT4O`, `OPENAI_GPT4O_MINI`, etc.
- Anthropic: `ANTHROPIC_CLAUDE4_OPUS`, `ANTHROPIC_CLAUDE4_SONNET`, `ANTHROPIC_CLAUDE4.5_SONNET`, etc.
- Azure: `AZURE_OPENAI_GPT5`, `AZURE_OPENAI_GPT4V`, etc.
- Gemini: `GEMINI_2.5_PRO`, `GEMINI_2.5_FLASH`, etc.
- AWS Bedrock: `BEDROCK_ANTHROPIC_CLAUDE4_OPUS_INFERENCE_PROFILE`, etc.
- Others: `INCEPTION_MERCURY_2`, `UI_TARS` (Doubao), Novita, Volcengine, OpenRouter

---

## Dependencies

Key external dependencies from `pyproject.toml`:

| Package | Purpose |
|---------|---------|
| `fastapi` | HTTP API framework |
| `playwright` | Browser automation |
| `sqlalchemy[mypy]` | ORM |
| `alembic` | DB migrations |
| `pydantic` + `pydantic-settings` | Data validation + config |
| `litellm` | Unified LLM API routing |
| `openai` | OpenAI client |
| `anthropic` | Anthropic client |
| `google-cloud-aiplatform` | Vertex AI (Gemini) |
| `aioboto3` | Async AWS SDK (S3, Secrets Manager) |
| `azure-storage-blob` | Azure Blob Storage |
| `azure-keyvault-secrets` | Azure Key Vault |
| `onepassword-sdk` | 1Password integration |
| `fastmcp` | MCP server framework |
| `typer` | CLI framework |
| `structlog` | Structured logging |
| `aioredlock` | Distributed locks (Redis) |
| `pyotp` | TOTP code generation |
| `pillow` | Image processing |
| `pypdf` + `pdfplumber` | PDF parsing |
| `tiktoken` | Token counting |
| `websockets` | WebSocket support |
| `posthog` | Analytics |
| `opentelemetry-api` | Distributed tracing |
| `croniter` | Cron expression parsing |

Frontend (key packages from `skyvern-frontend/package.json`):
- React 18, TypeScript, Vite
- React Router v6 (routing)
- TanStack Query v5 (data fetching)
- @xyflow/react (workflow canvas/editor)
- Radix UI (accessible components)
- Tailwind CSS (styling)
- Axios (HTTP client)
- CodeMirror (code editor in workflow builder)
- noVNC (VNC streaming in debug sessions)
- zustand (state management)

---

## Build & Run

### Backend
```bash
# Install Python dependencies
uv sync

# Run everything (API + UI)
skyvern run all

# Run API server only
skyvern run server

# Run UI only
skyvern run ui

# First-time setup
skyvern quickstart

# Database migrations
alembic upgrade head
alembic revision --autogenerate -m "description"

# Tests
pytest tests/

# Lint + format
ruff check && ruff format
mypy skyvern
```

### Frontend
```bash
cd skyvern-frontend
npm install
npm run dev        # Development (Vite dev server)
npm run build      # Production build
npm run lint
npm run format
```

### Docker
```bash
# Full stack with Docker Compose
pip install skyvern && skyvern quickstart
# OR
docker compose up -d
```

Server ports:
- `8000` — FastAPI API
- `6080` — VNC WebSocket streaming
- `9222` — CDP browser debugging

### SDK (Python)
```python
from skyvern import Skyvern
client = Skyvern(api_key="YOUR_API_KEY")  # defaults to SkyvernEnvironment.CLOUD

# Run a task
run = client.run_task(prompt="Search for...", url="https://example.com")

# Wait for completion
result = client.get_run(run.run_id)
```

---

## Patterns & Conventions

### Python Code Style
- Ruff for linting and formatting (configured in `pyproject.toml`)
- Line length: 120 characters
- Type hints everywhere; `async/await` for all I/O
- Structured logging via `structlog` (not bare `print` or `logging`)
- Pydantic v2 for all data models
- `StrEnum` for all enumerations
- `from __future__ import annotations` for deferred type evaluation

### API Design
- Current API prefix: `/v1/` (registered via `base_router`)
- Legacy API prefix: `/api/v1/` (registered via `legacy_base_router`) — same endpoints, `include_in_schema=False`
- All endpoints have trailing-slash duplicates as aliases (`include_in_schema=False`)
- Authentication via `Depends(org_auth_service.get_current_org)` on every protected endpoint
- `x-fern-sdk-method-name` OpenAPI extension maps endpoints to SDK method names
- `x-fern-examples` provides code samples for SDK docs

### Database Patterns
- All DB access goes through `AgentDB` (`skyvern/forge/sdk/db/agent_db.py`)
- Async SQLAlchemy with `asyncpg` (psycopg on older Python)
- IDs are generated with `generate_*_id()` functions that create prefixed UUIDs
- `created_at` / `modified_at` on all models (UTC datetimes)
- `deleted_at` for soft deletes (Folders, Workflows, Schedules, DebugSessions)
- Alembic migrations in `/alembic/versions/`, named `YYYY_MM_DD_HHMM-hash_description.py`

### Error Handling
- `SkyvernHTTPException` for user-facing HTTP errors (with message + status_code)
- Domain-specific exceptions in `skyvern/exceptions.py` and `skyvern/errors/`
- Exception handlers registered in `create_api_app()` in `api_app.py`
- Retry logic via `skyvern/forge/sdk/core/retry.py`

### Webhook Signature Verification
- Payload signed with HMAC-SHA256 using the organization's API key
- Headers: `x-skyvern-timestamp` + `x-skyvern-signature`
- Verify by generating signature on payload + timestamp, comparing

### Workflow YAML Format
- Workflows can be defined in YAML with `block_type`, `label`, `parameter_keys`, `url`, `navigation_goal`, `data_extraction_goal`
- `WorkflowDefinitionConverter` parses YAML → Python block objects
- `workflow_permanent_id` tracks identity across versions; `workflow_id` is version-specific

### MCP Server
- Exposed at `/mcp/` (FastMCP)
- 35+ tools across 6 categories (browser, actions, extraction, validation, credentials, workflows)
- Used by Claude Desktop, Cursor, Windsurf, and other AI clients
- Config at `skyvern/cli/mcp_tools/` — each file implements a category of tools

### Frontend Conventions
- Pages in `skyvern-frontend/src/routes/` (organized by feature area)
- API calls centralized in `skyvern-frontend/src/api/`
- TanStack Query for server state (caching, polling, mutations)
- React Router v6 with nested layouts
- Radix UI primitives + Tailwind CSS for styling
- No Redux; local state via useState/useReducer, server state via React Query

---

## Open Questions

See `context/open-questions.md`.
