# Context Index — Skyvern

## Overview

**Skyvern** is an AI-powered browser automation platform that automates browser-based workflows using LLMs and computer vision. Rather than relying on brittle XPath selectors or DOM parsing, Skyvern uses Vision LLMs to comprehend websites and intelligently execute actions. The system provides both a no-code workflow builder UI and a Playwright-compatible Python/TypeScript SDK for developers.

**Core Stack:**
- **Backend:** Python 3.11+ with FastAPI, SQLAlchemy, Playwright
- **Frontend:** React 18 + TypeScript with Vite, TailwindCSS, React Router, TanStack Query
- **Database:** PostgreSQL with Alembic migrations
- **Key Libraries:** Pydantic, Uvicorn, LiteLLM (multi-LLM provider), Redis, Anthropic SDK
- **TypeScript Client:** Published as `@skyvern/client` NPM package

## File Tree

```
context/skyvern/
├── skyvern/                          # Main Python backend package
│   ├── __init__.py                   # Lazy-loaded public API
│   ├── config.py                     # Settings/environment configuration (24KB)
│   ├── constants.py                  # Project constants
│   ├── cli/                          # CLI commands and utilities
│   │   ├── core/                     # Core CLI infrastructure
│   │   ├── commands/                 # CLI command implementations
│   │   └── mcp.py                    # Model Context Protocol integration
│   ├── webeye/                       # Browser interaction & automation core
│   │   ├── actions/                  # Action system (click, input, navigate)
│   │   ├── browser_manager.py        # Browser lifecycle management
│   │   ├── browser_state.py          # State tracking
│   │   └── real_browser_manager.py   # Real browser management
│   ├── forge/                        # Main API application
│   │   ├── api_app.py                # FastAPI app initialization
│   │   ├── agent.py                  # LLM agent orchestration
│   │   ├── forge_app.py              # App factory
│   │   ├── sdk/                      # Backend SDK
│   │   │   ├── api/                  # API utilities
│   │   │   ├── db/models.py          # SQLAlchemy ORM models
│   │   │   └── routes/               # API endpoint handlers
│   │   │       ├── agent_protocol.py # Task/workflow endpoints
│   │   │       ├── scripts.py        # Script CRUD
│   │   │       ├── credentials.py    # Secret management
│   │   │       └── streaming/        # WebSocket/SSE handlers
│   │   └── core/                     # Core SDK services
│   ├── services/                     # Business logic
│   │   ├── script_service.py         # Script generation
│   │   ├── task_v2_service.py        # Task orchestration
│   │   └── workflow_service.py       # Workflow management
│   └── schemas/                      # Pydantic models
│
├── skyvern-frontend/                 # React frontend
│   ├── src/
│   │   ├── api/                      # API client layer
│   │   ├── components/               # Reusable components
│   │   └── routes/                   # Page-level components
│
├── skyvern-ts/client/                # TypeScript SDK
│   ├── src/
│   │   ├── Client.ts                 # Main client class
│   │   └── api/                      # API resources
│
├── tests/                            # Test suites
├── alembic/                          # Database migrations
├── docker-compose.yml
├── Dockerfile
└── README.md
```

## Architecture

### System Design

Skyvern uses a **Vision LLM + Browser Automation** architecture:

1. **Web Comprehension:** Vision LLM analyzes screenshots to understand page structure
2. **Action Planning:** Agent reasons about required actions to achieve goal
3. **Browser Execution:** Playwright executes planned actions
4. **Feedback Loop:** Results feed back into agent for next iteration

### API Layer Architecture

```
User Interfaces (Web UI, Python SDK, TypeScript SDK)
            ↓
FastAPI Application (api_app.py)
  ├─ CORS, Auth, Error Handling Middleware
  ├─ base_router (main routes)
  ├─ legacy_base_router, legacy_v2_router (backward compat)
            ↓
Routes Layer (routes/)
  ├─ agent_protocol.py (tasks, workflows, runs)
  ├─ scripts.py (script CRUD)
  ├─ credentials.py (secrets)
  ├─ browser_sessions.py (browser lifecycle)
  ├─ streaming/ (WebSocket/SSE)
  └─ ... other endpoints
            ↓
Services Layer (services/)
  ├─ task_v2_service.py (task orchestration)
  ├─ script_service.py (script generation)
  ├─ workflow_service.py (workflow execution)
  └─ ... other services
            ↓
Core Layer (webeye/, agent.py, LLM APIs)
  ├─ Browser management (Playwright)
  ├─ LLM agents (multiple providers via LiteLLM)
  └─ Action execution
            ↓
Data Layer (SQLAlchemy, PostgreSQL, S3/Azure Blob)
```

## Key Concepts

### Tasks
- **Definition:** Single unit of work — navigate to URL, fill forms, extract data
- **Defined in:** `forge/sdk/db/models.py:TaskModel` (line ~85-160)
- **Execution:** Handled by `services/task_v2_service.py`
- **Statuses:** created, queued, running, completed, failed, terminated, timed_out, canceled, skipped, paused

### Workflows
- **Definition:** DAG of blocks with conditional branching and templating
- **Defined in:** `schemas/workflows.py:Workflow` (Pydantic model)
- **Storage:** YAML/JSON in database
- **Blocks:** Task, Script, Action, Condition, Loop, Code, Prompt, Webhook
- **Templating:** Jinja2 support for dynamic parameters
- **Execution:** `services/workflow_service.py:WorkflowService`
- **Runs:** Each execution tracked as `WorkflowRun`

### Scripts
- **Definition:** Reusable Python or TypeScript automation code
- **Generation:** `services/script_service.py:generate_script()`
- **Review:** `services/script_reviewer.py:ScriptReviewer`
- **Execution:** Standalone or as part of workflow
- **Caching:** Results cached for quick replay

### Browser Sessions
- **Definition:** Persistent browser state (cookies, storage, etc.)
- **Types:** Temporary (task-scoped) or persistent (saved/reloaded)
- **Storage:** Session data in S3/blob storage
- **Streaming:** VNC/CDP for real-time observation

### Artifacts
- **Definition:** Outputs from execution (screenshots, logs, extracted data)
- **Storage:** S3 or Azure Blob Storage (configurable via `SKYVERN_STORAGE_TYPE`)
- **Types:** recording, screenshot_action, screenshot_llm, llm_response, html_scrape, pdf, etc.
- **Access:** Signed URLs for temporary public access

### Credentials
- **Definition:** Secrets (API keys, passwords, OAuth, OTP)
- **Storage:** Encrypted in database
- **Sync:** Optional vault integration (OnePassword, Bitwarden, Azure Key Vault)
- **Usage:** Available as variables in workflows/scripts
- **Handler:** `routes/credentials.py`, `services/` integration

### Actions
- **Definition:** Low-level browser interactions (click, input, scroll, navigate, etc.)
- **Types:** Defined in `webeye/actions/action_types.py`
- **Handler:** `webeye/actions/handler.py` executes via Playwright
- **Caching:** Previous actions replayable for performance

## API Surface

All endpoints in `skyvern/forge/sdk/routes/`. Main router in `api_app.py` as `base_router`.

### Core Endpoints (v1 API)

#### Tasks
- `POST /run/tasks` — Create and run task
  - File: `agent_protocol.py:run_task_handler()`
  - Request: `TaskRequest` (goal, URL, extraction schema)
  - Response: `TaskResponse` (task_id, status)

- `GET /runs/{run_id}` — Get task/run details
  - File: `agent_protocol.py:get_run()`
  - Response: `RunResponse` (status, steps, artifacts)

#### Workflows
- `POST /workflows` — Create workflow
  - File: `agent_protocol.py:create_workflow()`
  - Request: `WorkflowCreateYAMLRequest`
  - Response: `Workflow` (workflow_id)

- `PUT /workflows/{workflow_id}` — Update workflow
  - File: `agent_protocol.py:update_workflow()`

- `GET /workflows` — List workflows
  - File: `agent_protocol.py:get_workflows()`
  - Query: folder_id, limit, offset, sort, order
  - Response: `List[Workflow]`

- `POST /workflows/{workflow_id}/run` — Execute workflow
  - File: `agent_protocol.py:run_workflow_handler()`
  - Request: `WorkflowRunRequest` (parameters, proxy, etc.)
  - Response: `WorkflowRunResponse` (workflow_run_id)

#### Scripts
- `POST /scripts` — Create script
  - File: `scripts.py:create_script()`
  - Request: `CreateScriptRequest`
  - Response: `ScriptResponse`

- `GET /scripts/{script_id}` — Get script
  - File: `scripts.py:get_script()`

- `POST /scripts/{script_id}/run` — Execute script
  - File: `scripts.py:run_script()`

#### Credentials
- `POST /credentials` — Create credential
  - File: `credentials.py:create_credential()`
  - Request: `CredentialCreateRequest`
  - Response: `CredentialResponse`

- `GET /credentials` — List credentials
  - File: `credentials.py:get_credentials()`

#### Browser Sessions
- `POST /browser-sessions` — Create session
  - File: `browser_sessions.py:create_browser_session()`
  - Response: `BrowserSessionResponse`

- Stream browser: WebSocket to `/browser-sessions/{session_id}/stream`
  - File: `streaming/channels/` (VNC, CDP handlers)

#### Run Blocks
- `POST /run-blocks` — Execute workflow block
  - File: `run_blocks.py:execute_run_block()`
  - Request: `BlockRunRequest`
  - Response: `BlockRunResponse`

#### Streaming
- `POST /event_source` — Subscribe to events
  - File: `event_source_stream.py:event_source_route()`
  - Response: Server-sent events stream

### Authentication
- **Method:** JWT Bearer token
- **Handler:** `internal_auth.py:verify_token()`
- **Flow:** Token → HTTP-only cookie (frontend) or header (SDK)
- **Algorithm:** HS256 (default), `SECRET_KEY` from config

## Data Model

### Core Models (forge/sdk/db/models.py)

**TaskModel** (line ~85-160)
- Fields: task_id, organization_id, status, url, navigation_goal, extracted_information
- Relationships: organization, workflow_run, browser_session
- Indexes: (organization_id, created_at), status

**WorkflowModel**
- Fields: workflow_permanent_id, workflow_id, organization_id, definition (JSON), status
- Relationships: organization, runs, versions

**WorkflowRunModel**
- Fields: workflow_run_id, workflow_id, organization_id, status, parameters
- Relationships: workflow, blocks

**ScriptModel**
- Fields: script_id, organization_id, code, language, status
- Relationships: organization, revisions, runs

**CredentialModel**
- Fields: credential_id, organization_id, type, encrypted_value
- Encryption: Fernet (via boto3/Azure)

**StepModel**
- Fields: step_id, task_id, index, status, llm_response, screenshot
- Relationships: task, artifacts, actions

**ActionModel**
- Fields: action_id, step_id, type, target_element, parameters
- Relationships: step, results

**ArtifactModel**
- Fields: artifact_id, task_id, step_id, artifact_type, uri, signed_url
- Storage: S3/Azure Blob paths

**BrowserSessionModel**
- Fields: browser_session_id, organization_id, status, proxy_location
- Relationships: organization

**OrganizationModel**
- Fields: organization_id, name, status, api_key_hash
- Relationships: users, tasks, workflows, credentials

### Pydantic Schemas (schemas/)

**Request Models:**
- `TaskRequest` (schemas/runs.py): url, navigation_goal, data_extraction_goal, proxy_location
- `WorkflowRunRequest`: parameters, proxy_location, engine
- `CreateWorkflowRequest`: definition (YAML/JSON), title
- `CredentialCreateRequest`: type, value, metadata

**Response Models:**
- `TaskResponse`: task_id, status, extracted_information, failure_reason
- `WorkflowRunResponse`: workflow_run_id, status, output
- `RunResponse`: status, steps, artifacts
- `ScriptResponse`: script_id, code, language, status

**Enums:**
- `TaskStatus`: created, queued, running, completed, failed, terminated, timed_out, canceled, skipped, paused
- `WorkflowRunStatus`: created, running, completed, failed, canceled
- `ArtifactType`: recording, screenshot_action, screenshot_llm, llm_response, html_scrape, pdf, etc.
- `BlockType`: task, script, action, condition, loop, code, prompt, webhook, etc.

## Auth & Middleware

### Authentication Flow
1. User logs in → JWT token issued
2. Token stored in HTTP-only cookie (frontend) or passed in header (SDK)
3. Request processing:
   - `internal_auth.py:verify_token()` extracts & validates JWT
   - Sets `organization_id` in request context
   - Org-level authorization (all members have full access)

### Middleware Chain (api_app.py)
- CORS middleware (configurable origins, default "*")
- RawContextMiddleware (ExecutionDatePlugin for timestamps)
- Request logging middleware (optional, controlled by `LOG_RAW_API_REQUESTS`)
- Custom error handling:
  - `SkyvernHTTPException` → formatted JSON
  - `ValidationError` → human-readable field errors

## Configuration

| Variable | Purpose | Default | File |
|----------|---------|---------|------|
| `ENV` | Environment (local, staging, prod) | local | .env |
| `PORT` | API server port | 8000 | config.py |
| `SECRET_KEY` | JWT signing key | PLACEHOLDER | config.py |
| `DATABASE_STRING` | PostgreSQL connection | postgresql+asyncpg://... | config.py |
| `REDIS_URL` | Redis connection | redis://localhost:6379/0 | config.py |
| `LOG_LEVEL` | Logging level (DEBUG, INFO, WARNING, ERROR) | INFO | config.py |
| `LOG_PATH` | Log file directory | ./log | config.py |
| `ARTIFACT_STORAGE_PATH` | Local artifact storage path | ./skyvern/artifacts | config.py |
| `SKYVERN_STORAGE_TYPE` | Storage backend (local, s3cloud, azureblob) | local | config.py |
| `AWS_REGION` | AWS region for S3 | us-east-1 | config.py |
| `AWS_S3_BUCKET_ARTIFACTS` | S3 bucket for artifacts | skyvern-artifacts | config.py |
| `BROWSER_TYPE` | Browser (chromium-headful, firefox, webkit) | chromium-headful | config.py |
| `BROWSER_WIDTH` | Viewport width | 1920 | config.py |
| `BROWSER_HEIGHT` | Viewport height | 1080 | config.py |
| `BROWSER_TIMEZONE` | Browser timezone | America/New_York | config.py |
| `BROWSER_LOCALE` | Browser locale | None | config.py |
| `BROWSER_ACTION_TIMEOUT_MS` | Action timeout | 5000 | config.py |
| `MAX_STEPS_PER_RUN` | Max task steps | 10 | config.py |
| `MAX_STEPS_PER_TASK_V2` | Max steps in task v2 | 25 | config.py |
| `MAX_RETRIES_PER_STEP` | Retry attempts | 5 | config.py |
| `LLM_KEY` | Active LLM model | OPENAI_GPT4O | config.py |
| `ENABLE_OPENAI` | Enable OpenAI | false | .env.example |
| `OPENAI_API_KEY` | OpenAI API key | "" | .env.example |
| `ENABLE_ANTHROPIC` | Enable Anthropic | false | .env.example |
| `ANTHROPIC_API_KEY` | Anthropic API key | "" | .env.example |
| `ENABLE_AZURE` | Enable Azure OpenAI | false | .env.example |
| `POSTHOG_PROJECT_API_KEY` | Analytics key | phc_bVT2ugnZhMHRWqMvSRHPdeTjaPxQqT3QSsI3r5FlQR5 | config.py |
| `SKYVERN_TELEMETRY` | Enable telemetry | true | config.py |
| `JSON_LOGGING` | Structured JSON logging | false | config.py |

## Dependencies

### Key Python Libraries

| Package | Version | Purpose |
|---------|---------|---------|
| FastAPI | >=0.121.0 | Web framework |
| Uvicorn | >=0.35.0 | ASGI server |
| Pydantic | >=2.5.2 | Data validation |
| SQLAlchemy | >=2.0.29 | ORM |
| Alembic | >=1.12.1 | Migrations |
| Playwright | >1.46.0 | Browser automation |
| LiteLLM | >=1.80.10 | Multi-LLM provider |
| Anthropic | >=0.50.0 | Anthropic API |
| OpenAI | >=1.68.2 | OpenAI API |
| Psycopg | 3.1.18 | PostgreSQL adapter |
| AsyncPG | >=0.30.0 | Async PostgreSQL |
| Structlog | >=23.2.0 | Structured logging |
| Typer | >=0.16.0 | CLI framework |
| Jinja2 | >=3.1.2 | Template engine |
| aiohttp | >=3.8.5 | Async HTTP |
| websockets | >=12.0 | WebSocket support |
| aioboto3 | >=14.3.0 | Async AWS SDK |
| azure-storage-blob | >=12.26.0 | Azure Blob Storage |
| posthog | >=3.7.0 | Analytics |

### Key Frontend Libraries

| Package | Version | Purpose |
|---------|---------|---------|
| React | ^18.2.0 | UI framework |
| React Router | ^6.30.2 | Routing |
| TanStack Query | ^5.28.6 | Server state |
| Zustand | ^4.5.2 | Client state |
| Axios | ^1.13.5 | HTTP client |
| Vite | ^5.4.21 | Build tool |
| TailwindCSS | ^3.4.17 | Styling |
| Radix UI | ^1.x | Components |
| React Hook Form | ^7.51.1 | Forms |
| React Flow | ^12.1.1 | Diagrams |

## Build & Run

### Local Development

```bash
# Install Python SDK
pip install skyvern

# Run quickstart (server + UI)
skyvern quickstart

# Or manually start services:
# Backend: python -m skyvern.cli.commands
# Frontend: cd skyvern-frontend && npm run dev
# Database: python -m alembic upgrade head
```

### Docker Compose

```bash
docker-compose up
# Starts: PostgreSQL, Redis, Backend (8000), Frontend (5173)
```

### Build Commands

**Backend:**
- `pytest` — Run tests
- `mypy skyvern/` — Type checking
- `python -m alembic upgrade head` — Migrate DB

**Frontend:**
- `npm run dev` — Dev server
- `npm run build` — Production build
- `npm run lint` — Linting
- `npm test` — Vitest

### Docker Images

- **Backend:** `Dockerfile` (Python 3.11, Playwright, Chrome)
- **Frontend:** `Dockerfile.ui` (Node 20, build artifacts)

## Patterns & Conventions

### Naming
- **IDs:** snake_case with prefix (task_abc123, workflow_xyz789)
- **Classes/Types:** PascalCase
- **Functions/vars:** snake_case
- **Constants:** UPPER_SNAKE_CASE
- **DB tables:** lowercase_plural
- **API routes:** kebab-case

### Error Handling
- Custom exceptions inherit from `SkyvernHTTPException`
- HTTP errors map to status codes (400, 404, 409, 500)
- Pydantic validation with formatted errors in middleware
- Structlog for logging: `LOG = structlog.get_logger()`

### Async/Await
- All DB queries: async SQLAlchemy (`AsyncSession`)
- All I/O: async/await (aiohttp, asyncpg, aiofiles)
- No blocking operations in async functions
- DB context: `SkyvernContext.get_agent_db()`

### Database
- **Lazy-loaded IDs:** Generator functions (generate_task_id, etc.)
- **Timestamps:** created_at, modified_at auto-populated
- **Soft deletes:** Some models use deleted_at flag
- **Relationships:** SQLAlchemy with ForeignKey constraints
- **Indexes:** Strategic on organization_id, status, created_at

### API Responses
- **Success:** Return Pydantic model (auto-JSON serialized)
- **Pagination:** limit, offset query params, returns List[Model]
- **Filtering:** Status, organization_id, etc. as query params
- **OpenAPI:** Auto-generated from routes, customizable with openapi_extra

### Frontend
- **Components:** Functional with hooks
- **State:** Zustand (global), React Query (server), form libraries
- **API calls:** Custom hooks (useGetWorkflow, useCreateTask, etc.)
- **Streaming:** SSE or WebSocket via custom hooks
- **Routing:** React Router with nested routes
- **Styling:** TailwindCSS + Radix UI

### Workflow Templating
- **Language:** Jinja2 with {{ }} for variables
- **Validation:** Block labels/parameter keys sanitized to valid identifiers
- **Scoping:** Block outputs as {block_label}_output in downstream blocks
- **Type safety:** Workflow definition validated at creation

### Testing
- **Unit tests:** `tests/unit/`, pytest + fixtures
- **SDK tests:** `tests/sdk/` for Python/TypeScript clients
- **Database:** In-memory or test DB (conftest.py)
- **Mocking:** External services mocked in unit tests

## Open Questions

1. **RBAC**: Are role-based access controls (admin, editor, viewer) planned?

2. **Database Replica**: `DATABASE_REPLICA_STRING` exists but unclear if actively used for read-write splitting.

3. **Workflow Triggering**: How are scheduled/event-triggered workflows handled?

4. **State Persistence**: State preservation for long-running workflows during server restarts?

5. **Concurrency Limits**: What's the max concurrent tasks/workflows? Is this configurable per org?

6. **LiteLLM Coverage**: Full extent of multi-provider support and test coverage?

7. **Workspace/Team**: Session.json has projectName — is there a workspace concept beyond organization?

8. **Artifact Cleanup**: Cleanup service triggers and retention policy details?

9. **WebSocket Auth**: Are streaming channel tokens different from API JWT?

10. **Browser Extensions**: Are browser extensions support (EXTENSIONS config) fully documented/used?

