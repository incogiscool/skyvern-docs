# Context Index

## Overview

Skyvern is an open-source AI-powered browser automation platform that uses LLMs and computer vision (Vision LLMs / Playwright) to automate web workflows without brittle XPath selectors. The core stack is Python 3.11+ (FastAPI backend), React/TypeScript frontend, PostgreSQL, Redis, and Playwright for browser control. Users interact via a REST/SDK API, a no-code web UI (app.skyvern.com), a CLI (`skyvern`), and an MCP server for AI assistant integration. There are three deployment modes: Skyvern Cloud, self-hosted Docker/Docker Compose, and local development (`pip install skyvern && skyvern quickstart`).

---

## File Tree

```
/source
├── alembic/                   # DB migrations (chronological, ~100+ files)
│   └── versions/
├── fern/                      # Fern-generated API docs + existing MDX pages
│   ├── docs.yml               # Fern nav/layout config
│   ├── generators.yml         # SDK generation config
│   ├── openapi/               # OpenAPI overrides
│   ├── introduction.mdx       # Root intro page
│   ├── getting-started/       # quickstart, prompting guide, etc.
│   ├── running-tasks/         # Tasks docs (run, visualize, cancel, webhooks, proxy, local browser)
│   ├── workflows/             # Workflow docs (manage, blocks, parameters, run, consistency)
│   ├── credentials/           # Cred docs (password, credit card, TOTP, Bitwarden, custom)
│   ├── browser-sessions/      # Browser sessions + profiles docs
│   ├── observability/         # Laminar tracing doc
│   └── integrations/          # MCP, CLI, Zapier, Make, n8n, Workato, Ollama docs
├── skyvern/                   # Main Python package
│   ├── __main__.py            # CLI entrypoint → cli_app()
│   ├── _cli_bootstrap.py      # Logging config for CLI
│   ├── _version.py
│   ├── config.py              # Pydantic Settings (all env vars)
│   ├── constants.py
│   ├── analytics.py
│   ├── exceptions.py
│   ├── schemas/               # Public Pydantic schemas (runs, workflows, artifacts, etc.)
│   │   ├── runs.py            # TaskRunRequest, WorkflowRunRequest, RunEngine, RunStatus, ProxyLocation
│   │   ├── workflows.py       # BlockType, YAML block types, parameter YAML types
│   │   ├── artifacts.py
│   │   ├── browser_sessions.py
│   │   ├── run_blocks.py
│   │   ├── steps.py
│   │   ├── webhooks.py
│   │   ├── folders.py
│   │   ├── scripts.py
│   │   └── docs/              # Doc strings and examples for API fields
│   ├── client/                # Fern-generated Python SDK client
│   │   ├── client.py          # Skyvern + AsyncSkyvern class with all SDK methods
│   │   ├── raw_client.py      # RawSkyvern (returns raw HTTP responses)
│   │   ├── environment.py     # SkyvernEnvironment enum (CLOUD, STAGING)
│   │   ├── core/              # httpx wrappers, SSE, pydantic utilities
│   │   ├── errors/            # Typed error classes
│   │   ├── types/             # Generated Pydantic models for all API types
│   │   └── scripts/           # Scripts sub-client
│   ├── cli/                   # Typer CLI commands
│   │   ├── commands/
│   │   │   ├── __init__.py    # cli_app Typer tree — all sub-typers registered here
│   │   │   └── browser.py     # browser_app (session + direct actions)
│   │   ├── run_commands.py    # `skyvern run` (server, mcp, ui)
│   │   ├── quickstart.py      # `skyvern quickstart`
│   │   ├── init_command.py    # `skyvern init`
│   │   ├── auth_command.py    # `skyvern login`
│   │   ├── setup_commands.py  # `skyvern setup` (MCP auto-config)
│   │   ├── workflow.py        # `skyvern workflow` commands
│   │   ├── tasks.py           # `skyvern tasks` commands
│   │   ├── credentials.py     # `skyvern credentials` commands
│   │   ├── mcp.py / mcp_commands.py  # MCP server launch
│   │   ├── mcp_tools/         # 35 MCP tool implementations
│   │   └── core/              # Browser launcher, session manager, ngrok, client helpers
│   ├── core/                  # Script generation utilities
│   │   └── script_generations/
│   ├── forge/                 # Server-side agent / workflow engine
│   │   ├── api_app.py         # FastAPI app factory (CORS, middleware, lifespan)
│   │   ├── forge_app.py       # Global app state (DATABASE, PERSISTENT_SESSIONS_MANAGER)
│   │   ├── agent.py           # Core agent logic
│   │   ├── agent_functions.py
│   │   ├── prompts.py         # LLM prompt templates
│   │   └── sdk/
│   │       ├── routes/        # All FastAPI route handlers
│   │       │   ├── agent_protocol.py      # Primary routes (tasks, workflows, artifacts, folders, etc.)
│   │       │   ├── browser_sessions.py    # Browser session endpoints
│   │       │   ├── browser_profiles.py    # Browser profile endpoints
│   │       │   ├── credentials.py         # Credential CRUD + TOTP
│   │       │   ├── scripts.py             # Scripts endpoints
│   │       │   ├── sdk.py                 # SDK action endpoint
│   │       │   ├── routers.py             # base_router, legacy_base_router, legacy_v2_router
│   │       │   └── streaming/             # WebSocket streaming (CDP, VNC, execution)
│   │       ├── db/
│   │       │   ├── models.py              # All SQLAlchemy ORM models
│   │       │   ├── agent_db.py            # DB access layer
│   │       │   ├── enums.py               # TaskType, WorkflowRunTriggerType, OrganizationAuthTokenType
│   │       │   └── id.py                  # ID generation (prefixed snowflake-style)
│   │       ├── schemas/                   # Server-side Pydantic schemas
│   │       │   ├── tasks.py               # TaskBase, TaskRequest, TaskStatus, Task, TaskResponse
│   │       │   ├── workflow_runs.py       # WorkflowRunBlock, WorkflowRunTimeline
│   │       │   ├── credentials.py         # CredentialType, PasswordCredential, CreditCardCredential, etc.
│   │       │   ├── persistent_browser_sessions.py  # PersistentBrowserSession, PersistentBrowserType
│   │       │   └── ...
│   │       ├── api/
│   │       │   ├── llm/
│   │       │   │   ├── config_registry.py # LLMConfigRegistry — registers all enabled LLM providers
│   │       │   │   ├── api_handler.py
│   │       │   │   └── models.py          # LLMConfig, LLMRouterConfig, LiteLLMParams
│   │       │   ├── aws.py / azure.py / real_azure.py  # Cloud integrations
│   │       │   └── custom_credential_client.py
│   │       ├── artifact/                  # Artifact storage (local, S3, Azure Blob)
│   │       │   ├── manager.py
│   │       │   ├── models.py              # ArtifactType, Artifact
│   │       │   └── storage/              # base, local, s3, azure, factory
│   │       ├── core/
│   │       │   ├── security.py            # JWT creation, webhook signature generation
│   │       │   ├── skyvern_context.py     # Request context (org, run IDs)
│   │       │   └── permissions/
│   │       ├── workflow/
│   │       │   ├── models/
│   │       │   │   ├── block.py           # All block classes (Block, ForLoopBlock, CodeBlock, etc.)
│   │       │   │   ├── parameter.py       # ParameterType, Parameter subclasses
│   │       │   │   └── workflow.py        # Workflow orchestration
│   │       │   ├── context_manager.py
│   │       │   └── service.py
│   │       └── services/
│   │           ├── org_auth_service.py    # get_current_org FastAPI dependency
│   │           └── local_org_auth_token_service.py
│   └── webeye/                # Browser interaction layer (Playwright)
├── skyvern-frontend/          # React/Vite frontend (TypeScript)
│   └── src/
│       ├── routes/            # Page-level route components
│       ├── components/        # Shared UI components
│       ├── api/               # API client calls
│       ├── hooks/
│       └── store/
├── skyvern-ts/client/         # Fern-generated TypeScript SDK
├── tests/
│   ├── unit/                  # Unit tests (pytest)
│   ├── smoke_tests/
│   └── sdk/python_sdk/        # SDK integration tests
├── alembic.ini
├── docker-compose.yml         # Postgres + Skyvern + UI services
├── Dockerfile / Dockerfile.ui
├── pyproject.toml             # Project metadata, deps, scripts
├── .env.example               # All configurable env vars with comments
└── README.md
```

---

## Architecture

### Data Flow

1. Client sends API request with `x-api-key` header → FastAPI (port 8000)
2. `org_auth_service.get_current_org` dependency resolves org from JWT token (DB lookup, TTL-cached 1h)
3. Route handler delegates to service layer (e.g., `run_service`, `workflow_service`)
4. Service layer creates DB records via `AgentDB` (SQLAlchemy async), queues work
5. Agent processes steps: Playwright browser action → screenshot → Vision LLM call → parse action → execute
6. Artifacts (screenshots, recordings, HTML, LLM prompts/responses) stored in artifact storage (local/S3/Azure)
7. On completion, webhook fired to `webhook_callback_url` if set

### Major Subsystems

- **API Server** (`skyvern/forge/api_app.py`): FastAPI app with CORS, structured logging, OpenTelemetry. Servers at `https://api.skyvern.com`, `http://localhost:8000`.
- **Agent Engine** (`skyvern/forge/agent.py`, `agent_functions.py`): Core task-running loop. Two engines: skyvern-1.0 (TaskV1) and skyvern-2.0 (TaskV2/ThoughtChain).
- **Workflow Engine** (`skyvern/forge/sdk/workflow/`): Block-based DAG runner. Blocks execute sequentially or in loops/branches.
- **LLM Layer** (`skyvern/forge/sdk/api/llm/`): LiteLLM-based abstraction. `LLMConfigRegistry` registers enabled providers at startup. Supports OpenAI, Anthropic, Azure, Gemini, Bedrock, Vertex AI, Ollama, Novita, Volcengine, UI-TARS.
- **Browser/Playwright** (`skyvern/webeye/`): Browser session management, page scraping, element tree extraction, action execution.
- **Persistent Browser Sessions** (`skyvern/forge/sdk/schemas/persistent_browser_sessions.py`, DB model at `models.py:L896`): Long-lived browser containers with CDP access.
- **Credentials** (`skyvern/forge/sdk/routes/credentials.py`): Stores and retrieves passwords, credit cards, TOTP secrets. Backends: Skyvern vault (DB + AES encrypt), Bitwarden, 1Password, Azure Key Vault, Custom HTTP service.
- **Artifact Storage** (`skyvern/forge/sdk/artifact/`): Factory selects local/S3/Azure based on `SKYVERN_STORAGE_TYPE`.
- **CLI** (`skyvern/cli/`): Typer-based CLI. Entry point: `skyvern/__main__.py → cli_app`.
- **MCP Server** (`skyvern/cli/mcp_tools/`): 35 tools across 6 categories exposed via FastMCP.
- **Python SDK** (`skyvern/client/client.py`): Fern-generated. `Skyvern` (sync) and `AsyncSkyvern` (async) classes.

---

## Key Concepts

- **Task** — Single browser automation goal with a `prompt` + optional `url`. Executed by an agent engine (skyvern-1.0 or skyvern-2.0).
  - Defined in: `skyvern/schemas/runs.py:L367` (TaskRunRequest), `skyvern/forge/sdk/schemas/tasks.py:L25` (TaskBase/Task)
  - DB model: `skyvern/forge/sdk/db/models.py:L74` (TaskModel)

- **Workflow** — Multi-step automation DAG composed of blocks. Has parameters, versioning (permanent ID), and can be scheduled.
  - Defined in: `skyvern/schemas/workflows.py`, `skyvern/forge/sdk/workflow/models/workflow.py`
  - DB models: `models.py:L276` (WorkflowModel), `L385` (WorkflowRunModel)

- **Block** — Unit of work in a workflow. All block types defined as `BlockType` enum.
  - `skyvern/schemas/workflows.py:L358` (BlockType enum)
  - Block classes: `skyvern/forge/sdk/workflow/models/block.py`
  - 25 block types: TASK, TaskV2, FOR_LOOP, CONDITIONAL, CODE, TEXT_PROMPT, DOWNLOAD_TO_S3, UPLOAD_TO_S3, FILE_UPLOAD, SEND_EMAIL, FILE_URL_PARSER, VALIDATION, ACTION, NAVIGATION, EXTRACTION, LOGIN, WAIT, FILE_DOWNLOAD, GOTO_URL, PDF_PARSER, HTTP_REQUEST, HUMAN_INTERACTION, PRINT_PAGE, WORKFLOW_TRIGGER

- **Run Engine** — Which agent powers execution.
  - `skyvern/schemas/runs.py:L341` (RunEngine enum)
  - Values: `skyvern-1.0`, `skyvern-2.0` (default), `openai-cua`, `anthropic-cua`, `ui-tars`

- **Run Status** — Lifecycle: `created → queued → running → completed|failed|terminated|timed_out|canceled`
  - `skyvern/schemas/runs.py:L353`

- **Parameter** — Workflow input placeholder. Types: WORKFLOW, CONTEXT, AWS_SECRET, BITWARDEN_*, ONEPASSWORD, AZURE_VAULT_CREDENTIAL, OUTPUT, CREDENTIAL, AZURE_SECRET
  - `skyvern/forge/sdk/workflow/models/parameter.py:L25` (ParameterType)

- **Proxy Location** — Enum of residential proxy regions.
  - `skyvern/schemas/runs.py:L36` (ProxyLocation), with GeoTarget class at `:L184`
  - Countries: US (default), AR, AU, BR, CA, DE, ES, FR, GB, IE, IN, IT, JP, MX, NL, NZ, PH, KR, TR, ZA + ISP pool

- **Artifact** — File output of a step (screenshots, recordings, HTML, LLM logs, etc.)
  - `skyvern/forge/sdk/artifact/models.py:L10` (ArtifactType), `skyvern/forge/sdk/db/models.py:L205` (ArtifactModel)

- **Browser Session** — Persistent cloud browser container for continuous task chaining.
  - `skyvern/forge/sdk/schemas/persistent_browser_sessions.py:L41` (PersistentBrowserSession)
  - DB model: `models.py:L896`

- **Browser Profile** — Snapshot of authenticated browser state for reuse across runs.
  - `skyvern/forge/sdk/schemas/browser_profiles.py`, DB model: `models.py:L940`

- **TOTP** — 2FA/MFA support. Codes sent to `/v1/credentials/totp` or polled from a `totp_url`.
  - DB model: `models.py:L702` (TOTPCodeModel)
  - Route: `skyvern/forge/sdk/routes/credentials.py:L119`

- **Credential** — Stored secret for login automation (password, credit card, or raw secret).
  - `skyvern/forge/sdk/schemas/credentials.py:L18` (CredentialType: password, credit_card, secret)
  - DB model: `models.py:L998`

---

## API Surface

All routes use prefix `/api/v1` (base_router) or legacy paths. Auth: `x-api-key` header (JWT token).

### Core Task/Run Endpoints (`skyvern/forge/sdk/routes/agent_protocol.py`)

| Method | Path | Description | Line |
|--------|------|-------------|------|
| POST | `/v1/run/tasks` | Run a task | L143 |
| POST | `/v1/run/workflows` | Run a workflow | L338 |
| GET | `/v1/runs/{run_id}` | Get run status & output | L445 |
| GET | `/v1/runs/{run_id}/timeline` | Get run timeline | L467 |
| POST | `/v1/runs/{run_id}/cancel` | Cancel a run | L487 |
| POST | `/v1/runs/{run_id}/retry_webhook` | Retry webhook | L1559 |
| GET | `/v1/runs/{run_id}/artifacts` | Get run artifacts | L1524 |
| GET | `/v1/artifacts/{artifact_id}` | Get single artifact | L1396 |
| POST | `/v1/run/tasks/login` | Login task shortcut | L570 |
| POST | `/v1/upload_file` | Upload a file | L3025 |

### Workflow Endpoints

| Method | Path | Description | Line |
|--------|------|-------------|------|
| GET | `/v1/workflows` | List workflows | L2665 |
| POST | `/v1/workflows` | Create workflow | L964 |
| GET | `/v1/workflows/{workflow_permanent_id}` | Get workflow | L2815 |
| GET | `/v1/workflows/{workflow_permanent_id}/versions` | Get versions | L2850 |
| POST | `/v1/workflows/{workflow_id}` | Update workflow | L987 |
| POST | `/v1/workflows/{workflow_id}/delete` | Delete workflow | L1049 |
| GET | `/v1/workflows/runs` | List workflow runs | L1445 |
| GET | `/v1/workflows/runs/{workflow_run_id}` | Get workflow run | L1615 |

### Folder Endpoints

| Method | Path | Line |
|--------|------|------|
| POST | `/v1/folders` | L1081 |
| GET | `/v1/folders/{folder_id}` | L1123 |
| GET | `/v1/folders` | L1167 |
| PUT | `/v1/folders/{folder_id}` | L1224 |
| DELETE | `/v1/folders/{folder_id}` | L1271 |
| PUT | `/v1/workflows/{workflow_permanent_id}/folder` | L1306 |

### Browser Sessions (`skyvern/forge/sdk/routes/browser_sessions.py`)

| Method | Path | Line |
|--------|------|------|
| GET | `/v1/browser_sessions` | L29 |
| POST | `/v1/browser_sessions` | L61 |
| POST | `/v1/browser_sessions/{browser_session_id}/close` | L115 |
| GET | `/v1/browser_sessions/{browser_session_id}` | L156 |

### Browser Profiles (`skyvern/forge/sdk/routes/browser_profiles.py`)

| Method | Path | Line |
|--------|------|------|
| POST | `/v1/browser_profiles` | L49 |
| GET | `/v1/browser_profiles` | L108 |
| GET | `/v1/browser_profiles/{profile_id}` | L159 |
| DELETE | `/v1/browser_profiles/{profile_id}` | L223 |

### Credentials (`skyvern/forge/sdk/routes/credentials.py`)

| Method | Path | Line |
|--------|------|------|
| POST | `/v1/credentials/totp` | L119 — send TOTP code |
| GET | `/v1/credentials/totp` | L198 — get TOTP codes |
| POST | `/v1/credentials` | L252 — create credential |
| GET | `/v1/credentials/{credential_id}` | L1305 |
| POST | `/v1/credentials/{credential_id}/update` | L1175 |
| POST | `/v1/credentials/{credential_id}/delete` | L1246 |
| GET | `/v1/credentials` | L1355 |

### Scripts (`skyvern/forge/sdk/routes/scripts.py`)

| Method | Path | Line |
|--------|------|------|
| POST | `/v1/scripts` | L206 |
| GET | `/v1/scripts/{script_id}` | L275 |
| GET | `/v1/scripts` | L234 |
| POST | `/v1/scripts/{script_id}/deploy` | L504 |

### SDK Action (`skyvern/forge/sdk/routes/sdk.py`)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/v1/sdk/run_action` | Run a single SDK browser action |

---

## Data Model

All ORM models in `skyvern/forge/sdk/db/models.py`. ID generation in `skyvern/forge/sdk/db/id.py` (prefixed snowflake-style).

### ID Prefixes

| Entity | Prefix | Example |
|--------|--------|---------|
| Task (v1) | `tsk` | `tsk_abc123` |
| Task (v2) | `tsk_v2` | `tsk_v2_abc123` |
| Workflow | `w` | `w_abc123` |
| Workflow permanent | `wpid` | `wpid_abc123` |
| Workflow run | `wr` | `wr_abc123` |
| Workflow run block | `wrb` | `wrb_abc123` |
| Step | `stp` | `stp_abc123` |
| Artifact | `a` | `a_abc123` |
| Browser session | `pbs` | `pbs_abc123` |
| Browser profile | `bp` | `bp_abc123` |
| Credential | `cred` | `cred_abc123` |
| Organization | `o` | `o_abc123` |
| Folder | `fld` | `fld_abc123` |
| Script | `s` | `s_abc123` |
| Debug session | `ds` | `ds_abc123` |

### Key DB Tables / Models

| Model | Table | Line (models.py) | Key Fields |
|-------|-------|------|-----------|
| TaskModel | tasks | L74 | task_id, organization_id, status, navigation_goal, url, extracted_information, error_code_mapping, max_steps_per_run, proxy_location, browser_session_id, model |
| StepModel | steps | L127 | step_id, task_id, status, output, order, retry_index, step_cost, input/output_token_count |
| WorkflowModel | workflows | L276 | workflow_id, workflow_permanent_id, organization_id, title, description, workflow_definition (JSON), model |
| WorkflowRunModel | workflow_runs | L385 | workflow_run_id, workflow_permanent_id, status, proxy_location, browser_session_id |
| WorkflowRunBlockModel | workflow_run_blocks | L757 | workflow_run_block_id, type, status, output, label |
| TaskV2Model | task_v2 | L822 | task_v2_id, url, prompt, status, max_steps, generate_script |
| ThoughtModel | observer_thoughts | L864 | thought_id, thought, observation, plan |
| PersistentBrowserSessionModel | persistent_browser_sessions | L896 | persistent_browser_session_id, status, timeout_minutes, proxy_location |
| BrowserProfileModel | browser_profiles | L940 | browser_profile_id, name |
| CredentialModel | credentials | L998 | credential_id, credential_type, credential (AES-encrypted JSON) |
| ArtifactModel | artifacts | L205 | artifact_id, artifact_type, uri, task_id, step_id, workflow_run_id |
| TOTPCodeModel | totp_codes | L702 | totp_code_id, totp_identifier, content, source |
| ActionModel | actions | L724 | action_id, action_type, confidence_float, created_by |
| TaskRunModel | task_runs | L957 | task_run_id, task_run_type, run_id |
| ScriptModel | scripts | L1059 | script_id, title, script_type |
| OrganizationModel | organizations | L158 | organization_id, organization_name, max_steps_per_run, domain |
| DebugSessionModel | debug_sessions | L1023 | debug_session_id, status, deleted_at |

---

## Auth & Middleware

### Authentication
- **Header**: `x-api-key: <JWT_TOKEN>` (primary) or `Authorization: Bearer <token>` (secondary)
- **FastAPI Dependency**: `org_auth_service.get_current_org` (`skyvern/forge/sdk/services/org_auth_service.py:L33`)
  - Verifies JWT via `settings.SECRET_KEY` / `settings.SIGNATURE_ALGORITHM`
  - Resolves org from `organization_auth_tokens` table
  - Result TTL-cached 1 hour (cachetools TTLCache, 128 entries)
- **Local dev**: Token stored in `.env` as `SKYVERN_API_KEY`. `regenerate_local_api_key()` in `local_org_auth_token_service.py` rotates it and writes `.env`.

### Webhook Signatures
- Signed with HMAC-SHA256 using the org's API key
- Headers sent: `x-skyvern-signature`, `x-skyvern-timestamp`
- Verification logic: `skyvern/forge/sdk/core/security.py:L44` (`generate_skyvern_webhook_signature`)

### Middleware Stack (`skyvern/forge/api_app.py`)
1. `RawContextMiddleware` (starlette-context) — sets execution date in request context
2. CORS middleware — configurable via `ALLOWED_ORIGINS` (default `["*"]`)
3. `log_raw_request_middleware` — structured logging of requests

---

## Configuration

All settings via `skyvern/config.py` (Pydantic Settings). Loaded from `.env`, `.env.staging`, `.env.prod`.

| Variable | Purpose | Default |
|----------|---------|---------|
| `ENV` | Environment name (local/staging/prod) | `local` |
| `DATABASE_STRING` | PostgreSQL connection string | `postgresql+psycopg://skyvern@localhost/skyvern` |
| `REDIS_URL` | Redis connection | `redis://localhost:6379/0` |
| `PORT` | API server port | `8000` |
| `LLM_KEY` | Active LLM model key | `OPENAI_GPT4O` |
| `SECONDARY_LLM_KEY` | Cheaper LLM for simple tasks | None |
| `ENABLE_OPENAI` | Enable OpenAI provider | `false` |
| `OPENAI_API_KEY` | OpenAI key | — |
| `ENABLE_ANTHROPIC` | Enable Anthropic | `false` |
| `ANTHROPIC_API_KEY` | Anthropic key | — |
| `ENABLE_AZURE` | Enable Azure OpenAI | `false` |
| `ENABLE_GEMINI` | Enable Gemini | `false` |
| `GEMINI_API_KEY` | Gemini key | — |
| `ENABLE_BEDROCK` | Enable AWS Bedrock | `false` |
| `SKYVERN_STORAGE_TYPE` | Artifact storage: `local`, `s3cloud`, `azureblob` | `local` |
| `AWS_REGION` | AWS region | `us-east-1` |
| `AWS_S3_BUCKET_ARTIFACTS` | S3 bucket for artifacts | `skyvern-artifacts` |
| `SKYVERN_API_KEY` | Local self-hosted API key | `PLACEHOLDER` |
| `SECRET_KEY` | JWT signing secret | `PLACEHOLDER` |
| `MAX_STEPS_PER_RUN` | Default max steps (v1) | `10` |
| `MAX_STEPS_PER_TASK_V2` | Default max steps (v2) | `25` |
| `BROWSER_TYPE` | `chromium-headful` or `chromium-headless` | `chromium-headful` |
| `VIDEO_PATH` | Recording output dir | `./videos` |
| `BITWARDEN_CLIENT_ID/SECRET` | Bitwarden integration | — |
| `OP_SERVICE_ACCOUNT_TOKEN` | 1Password service account | — |
| `CREDENTIAL_VAULT_TYPE` | Where creds stored: `bitwarden` or `azure_vault` | `bitwarden` |
| `CUSTOM_CREDENTIAL_API_BASE_URL` | Custom credential service | — |
| `ENABLE_LOG_ARTIFACTS` | Save LLM logs as artifacts | `false` |
| `ENABLE_CODE_BLOCK` | Allow Code Block in workflows | `true` |
| `WORKFLOW_TEMPLATING_STRICTNESS` | Jinja template mode: `strict` or `lax` | `lax` |

### LLM Provider Keys (registered in `config_registry.py`)
OpenAI: `OPENAI_GPT5`, `OPENAI_GPT4O`, `OPENAI_GPT4_TURBO`, etc.
Anthropic: `ANTHROPIC_CLAUDE4.5_SONNET`, `ANTHROPIC_CLAUDE4_SONNET`, `ANTHROPIC_CLAUDE3.7_SONNET`, etc.
Azure: `AZURE_OPENAI_GPT4V`, `AZURE_OPENAI_GPT5`, etc.
Gemini/Vertex: `GEMINI_2.5_PRO`, `VERTEX_GEMINI_2.5_PRO`, etc.
Bedrock: `BEDROCK_ANTHROPIC_CLAUDE4_SONNET`, etc.
Others: `OLLAMA`, `NOVITA_*`, `VOLCENGINE_*`, `UI_TARS_*`

---

## Python SDK

**Package**: `skyvern` (installed via `pip install skyvern`)  
**Client classes**: `Skyvern` (sync) / `AsyncSkyvern` (async) in `skyvern/client/client.py`

### Key SDK Methods

| Method | Description | Line |
|--------|-------------|------|
| `run_task(prompt, url, ...)` | Submit a task | L140 (sync), L2676 (async) |
| `run_workflow(workflow_id, ...)` | Run a workflow | L316, L2860 |
| `get_run(run_id)` | Poll run status | L468, L3020 |
| `cancel_run(run_id)` | Cancel a run | L499, L3059 |
| `get_run_artifacts(run_id)` | Get artifacts | L785, L... |
| `get_run_timeline(run_id)` | Get step timeline | L866 |
| `retry_run_webhook(run_id)` | Replay webhook | L826 |
| `get_workflows()` | List workflows | L532 |
| `create_workflow(yaml)` | Create workflow | L624 |
| `update_workflow(id, yaml)` | Update workflow | L673 |
| `delete_workflow(id)` | Delete workflow | L722 |
| `get_workflow(id)` | Get workflow | L988 |
| `get_workflow_versions(id)` | Workflow versions | L1031 |
| `get_workflow_runs(...)` | List workflow runs | L899 |
| `create_browser_session(...)` | Start a browser session | L1283 |
| `close_browser_session(id)` | Close browser session | L1370 |
| `get_browser_session(id)` | Get browser session | L1403 |
| `get_browser_sessions()` | List sessions | L1255 |
| `send_totp_code(...)` | Submit TOTP/2FA code | L1436 |
| `get_credentials()` | List credentials | L1511 |
| `create_credential(...)` | Create credential | L1552 |
| `update_credential(id, ...)` | Update credential | L1604 |
| `delete_credential(id)` | Delete credential | L1664 |
| `get_credential(id)` | Get credential | L1694 |
| `list_browser_profiles()` | List profiles | L1101 |
| `create_browser_profile(...)` | Create profile | L1136 |
| `get_browser_profile(id)` | Get profile | L1190 |
| `delete_browser_profile(id)` | Delete profile | L1223 |
| `upload_file(...)` | Upload a file | L1072 |
| `get_folders()` | List folders | L2336 |
| `create_folder(...)` | Create folder | L2384 |
| `update_workflow_folder(...)` | Move workflow to folder | L2546 |
| `login(...)` | Authenticate | L1727 |
| `download_files(...)` | Download task files | L1866 |
| `get_scripts()` | List scripts | L1970 |
| `create_script(...)` | Create script | L2011 |
| `deploy_script(id)` | Deploy script | L2086 |
| `run_sdk_action(action)` | Run single browser action | L2138 |

### SDK Instantiation

```python
from skyvern import Skyvern

# Cloud
client = Skyvern(api_key="YOUR_API_KEY")

# Self-hosted
client = Skyvern(base_url="http://localhost:8000", api_key="YOUR_LOCAL_KEY")

# Async
from skyvern import AsyncSkyvern
client = AsyncSkyvern(api_key="YOUR_API_KEY")
```

---

## CLI Commands

Entry: `skyvern` (installed script → `skyvern/__main__.py`)  
CLI tree registered in `skyvern/cli/commands/__init__.py`.

| Command | Description | File |
|---------|-------------|------|
| `skyvern login` | Authenticate, save API key | `cli/auth_command.py` |
| `skyvern quickstart` | One-command setup + start | `cli/quickstart.py` |
| `skyvern init` | Interactive config wizard | `cli/init_command.py` |
| `skyvern setup` | Auto-configure MCP for AI tools | `cli/setup_commands.py` |
| `skyvern run server` | Start API server | `cli/run_commands.py` |
| `skyvern run mcp [--http]` | Start MCP server | `cli/mcp_commands.py` |
| `skyvern status` | Check running services | `cli/status.py` |
| `skyvern stop` | Stop services | `cli/stop_commands.py` |
| `skyvern browser session create/list/close/get/connect` | Browser session mgmt | `cli/commands/browser.py` |
| `skyvern browser serve [--tunnel]` | Serve local browser for CDP | `cli/core/unified_server.py` |
| `skyvern browser navigate/act/extract/validate/click/type/...` | Browser automation | `cli/commands/browser.py` |
| `skyvern workflow list/get/run/delete` | Workflow management | `cli/workflow.py` |
| `skyvern tasks list/get` | Task management | `cli/tasks.py` |
| `skyvern credentials add/list/get/delete` | Credential management | `cli/credentials.py` |
| `skyvern block schema/validate` | Block schema inspection | `cli/block.py` |
| `skyvern skill` | Manage skill reference files | `cli/skill_commands.py` |

---

## MCP Server

The MCP server exposes **35 tools** in 6 categories. Implementation: `skyvern/cli/mcp_tools/`.

| Category | Tools |
|----------|-------|
| Browser sessions | `skyvern_browser_session_create/close/list/get/connect` |
| Browser actions | `skyvern_act/navigate/click/type/hover/scroll/select_option/press_key/wait` |
| Data extraction | `skyvern_extract/screenshot/evaluate` |
| Validation | `skyvern_validate` |
| Credentials & login | `skyvern_login/credential_list/credential_get/credential_delete` |
| Workflows | `skyvern_workflow_create/run/status/get/update/delete/cancel` + others |

MCP transports: stdio (`skyvern run mcp`) and HTTP (`skyvern run mcp --http`).

Cloud: configured at `https://api.skyvern.com/mcp/` with `x-api-key` header.

---

## Workflow Blocks (25 types)

Defined in `skyvern/schemas/workflows.py:L358` and implemented in `skyvern/forge/sdk/workflow/models/block.py`.

| Block | Class (block.py line) | Purpose |
|-------|----------------------|---------|
| TASK | `TaskBlock:L1191` | Navigate + extract (v1 engine) |
| TaskV2 | `TaskV2Block:L4433` | Navigate + extract (v2 engine) |
| FOR_LOOP | `ForLoopBlock:L1305` | Iterate over a list |
| CONDITIONAL | `ConditionalBlock:L5911` | Branch based on conditions |
| CODE | `CodeBlock:L2203` | Run custom Python/Playwright |
| TEXT_PROMPT | `TextPromptBlock:L2496` | LLM text-only prompt |
| DOWNLOAD_TO_S3 | `DownloadToS3Block:L2678` | Download file → S3 |
| UPLOAD_TO_S3 | `UploadToS3Block:L2766` | Upload to S3 |
| FILE_UPLOAD | `FileUploadBlock:L2877` | Upload to destination |
| SEND_EMAIL | `SendEmailBlock:L3128` | Send email |
| FILE_URL_PARSER | `FileParserBlock:L3465` | Parse PDF/CSV/Excel |
| PDF_PARSER | `PDFParserBlock:L3965` | Parse PDF |
| VALIDATION | `ValidationBlock:L4353` | Assert page state |
| ACTION | `ActionBlock:L4392` | Single action |
| NAVIGATION | `NavigationBlock:L4398` | Navigation only |
| EXTRACTION | `ExtractionBlock:L4406` | Data extraction only |
| LOGIN | `LoginBlock:L4414` | Login automation |
| WAIT | `WaitBlock:L4071` | Wait N seconds |
| FILE_DOWNLOAD | `FileDownloadBlock:L4420` | Download file |
| GOTO_URL | `UrlBlock:L4426` | Navigate to URL |
| HTTP_REQUEST | `HttpRequestBlock:L4646` | HTTP API call |
| HUMAN_INTERACTION | `HumanInteractionBlock:L4118` | Pause for human |
| PRINT_PAGE | `PrintPageBlock:L5125` | Print/PDF page |
| WORKFLOW_TRIGGER | `WorkflowTriggerBlock:L6467` | Trigger another workflow |

---

## Credential System

Three storage backends, configured by `CREDENTIAL_VAULT_TYPE`:

| Backend | Config | Handler |
|---------|--------|---------|
| Skyvern vault (DB) | Default | AES-encrypted in `credentials` table |
| Bitwarden | `BITWARDEN_CLIENT_ID`, `BITWARDEN_CLIENT_SECRET`, `BITWARDEN_MASTER_PASSWORD` | bitwarden-cli-server |
| 1Password | `OP_SERVICE_ACCOUNT_TOKEN` | onepassword-sdk |
| Azure Key Vault | `AZURE_TENANT_ID`, `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`, `AZURE_CREDENTIAL_VAULT` | azure-keyvault-secrets |
| Custom HTTP | `CUSTOM_CREDENTIAL_API_BASE_URL`, `CUSTOM_CREDENTIAL_API_TOKEN` | custom_credential_client.py |

Credential types: `password`, `credit_card`, `secret`
TOTP types: `authenticator`, `email`, `text` (phone), `magic_link`

---

## Existing Docs (fern/)

| Section | Files | Path |
|---------|-------|------|
| Introduction | `introduction.mdx` | `/fern/introduction.mdx` |
| Quickstart | `quickstart.mdx` | `/fern/getting-started/quickstart.mdx` |
| Skyvern in Action | `skyvern-in-action.mdx` | `/fern/getting-started/skyvern-in-action.mdx` |
| Prompting Guide | `prompting-guide.mdx` | `/fern/getting-started/prompting-guide.mdx` |
| Run Tasks | `run-tasks.mdx` | `/fern/running-tasks/run-tasks.mdx` |
| Visualizing Results | `visualizing-results.mdx` | `/fern/running-tasks/visualizing-results.mdx` |
| Cancel Runs | `cancel-runs.mdx` | `/fern/running-tasks/cancel-runs.mdx` |
| Webhooks FAQ | `webhooks-faq.mdx` | `/fern/running-tasks/webhooks-faq.mdx` |
| Local Browser | `connect-local-browser.mdx` | `/fern/running-tasks/connect-local-browser.mdx` |
| Proxy & Geo | `proxy-location.mdx` | `/fern/running-tasks/proxy-location.mdx` |
| Manage Workflows | `manage-workflows.mdx` | `/fern/workflows/manage-workflows.mdx` |
| Workflow Blocks | `workflow-blocks-details.mdx` | `/fern/workflows/workflow-blocks-details.mdx` |
| Workflow Parameters | `workflow-parameters.mdx` | `/fern/workflows/workflow-parameters.mdx` |
| Run Workflows | `run-workflows.mdx` | `/fern/workflows/run-workflows.mdx` |
| Consistent Workflows | `consistent-workflows.mdx` | `/fern/workflows/consistent-workflows.mdx` |
| Credentials Intro | `introduction.mdx` | `/fern/credentials/introduction.mdx` |
| Passwords | `passwords.mdx` | `/fern/credentials/passwords.mdx` |
| Credit Cards | `credit-cards.mdx` | `/fern/credentials/credit-cards.mdx` |
| TOTP/2FA | `totp.mdx` | `/fern/credentials/totp.mdx` |
| Bitwarden | `bitwarden.mdx` | `/fern/credentials/bitwarden.mdx` |
| Custom Credential Service | `custom-credential-service.mdx` | `/fern/credentials/custom-credential-service.mdx` |
| Browser Sessions Intro | `introduction.mdx` | `/fern/browser-sessions/introduction.mdx` |
| Browser Profiles | `browser-profiles.mdx` | `/fern/browser-sessions/browser-profiles.mdx` |
| Observability | `overview.mdx` | `/fern/observability/overview.mdx` |
| MCP Server | `mcp.mdx` | `/fern/integrations/mcp.mdx` |
| CLI & Skills | `cli.mdx` | `/fern/integrations/cli.mdx` |
| Zapier | `zapier.mdx` | `/fern/integrations/zapier.mdx` |
| Make | `make.com.mdx` | `/fern/integrations/make.com.mdx` |
| N8N | `n8n.mdx` | `/fern/integrations/n8n.mdx` |
| Workato | `workato.mdx` | `/fern/integrations/workato.mdx` |
| Ollama + LiteLLM | `ollama-litellm.mdx` | `/fern/integrations/ollama-litellm.mdx` |

---

## Dependencies

| Package | Purpose |
|---------|---------|
| `fastapi` | REST API framework |
| `playwright` | Browser automation |
| `sqlalchemy[mypy]` | ORM + async DB |
| `alembic` | DB migrations |
| `pydantic` / `pydantic-settings` | Data validation & settings |
| `litellm` | Unified LLM provider abstraction |
| `anthropic` | Direct Anthropic SDK |
| `openai` | OpenAI SDK |
| `google-cloud-aiplatform` | Vertex AI / Gemini |
| `aioboto3` | Async AWS (S3, Bedrock) |
| `azure-storage-blob` | Azure Blob Storage |
| `azure-keyvault-secrets` | Azure Key Vault |
| `onepassword-sdk` | 1Password integration |
| `typer` | CLI framework |
| `fastmcp` | MCP server framework |
| `structlog` | Structured logging |
| `PyJWT` | JWT auth |
| `aioredlock` | Distributed locking (Redis) |
| `jinja2` | Template rendering for workflow parameters |
| `pypdf` / `pdfplumber` | PDF parsing |
| `httpx` | Async HTTP client |
| `websockets` | WebSocket connections |
| `sse-starlette` | Server-sent events |
| `posthog` | Analytics |
| `opentelemetry-api` | Distributed tracing |
| `croniter` | Cron schedule parsing |

---

## Build & Run

### Quick Start (Cloud API)
```bash
pip install skyvern
# Use Skyvern Cloud
python -c "from skyvern import Skyvern; ..."
```

### Local Development
```bash
pip install skyvern
skyvern quickstart        # Interactive setup (Postgres + env + LLM)
# or
skyvern init              # Setup only
skyvern run server        # Start API server at :8000
```

### Docker Compose
```bash
pip install skyvern
skyvern quickstart        # Pulls images, starts all services
# or manual:
docker compose up -d
```

### DB Migrations
```bash
alembic upgrade head
```

### Tests
```bash
pytest tests/unit/        # Unit tests
pytest tests/sdk/         # SDK integration tests (requires running server)
```

### Frontend (dev)
```bash
cd skyvern-frontend
npm install
npm run dev               # Starts at :5173
npm run artifact-server   # Artifact file server
```

---

## Patterns & Conventions

### Code Style
- Python 3.11+, type annotations throughout
- Pydantic v2 for all schemas (`BaseModel`, `ConfigDict`)
- `StrEnum` for all enumerations (string-valued)
- Structlog for structured logging (`LOG = structlog.get_logger()`)
- Async-first: all DB calls, LLM calls, and browser operations are async
- `from __future__ import annotations` in most files

### ID Generation
- Custom snowflake-style IDs with string prefixes
- All `generate_*_id()` functions in `skyvern/forge/sdk/db/id.py`
- Format: `{prefix}_{base62_int}`

### Route Convention
- Canonical routes use `base_router` (included in schema)
- Legacy routes use `legacy_base_router` or `legacy_v2_router` (`include_in_schema=False`)
- Every canonical route has a trailing-slash alias marked `include_in_schema=False`
- Routes always `Depends(org_auth_service.get_current_org)` for auth

### Schema Organization
- Public API schemas: `skyvern/schemas/` (used by SDK and routes)
- Server-only schemas: `skyvern/forge/sdk/schemas/`
- ORM models: `skyvern/forge/sdk/db/models.py`
- Generated SDK types: `skyvern/client/types/`

### Workflow Parameters
- Jinja2 syntax: `{{parameter_key}}`
- Reserved keys: `current_item`, `current_value`, `current_index`, `current_date`, `workflow_title`, `workflow_id`, `workflow_permanent_id`, `workflow_run_id`, `workflow_run_outputs`, `workflow_run_summary`
- Block output referenced as `{{block_label}}` or `{{block_label_output}}`

### Error Handling
- FastAPI `HTTPException` for HTTP errors
- `SkyvernHTTPException` for custom Skyvern errors
- `error_code_mapping` parameter maps task failure reasons to custom error codes
- Webhook payload includes `failure_reason` field

### Webhook Signature
- HMAC-SHA256 of request body with org API key
- Headers: `x-skyvern-signature`, `x-skyvern-timestamp`
- Verify in `skyvern/forge/sdk/core/security.py`

---

## Open Questions

See `context/open-questions.md` for details.

---

## Important: Two `Skyvern` Classes

There are two different `Skyvern` classes in the package:

### 1. `from skyvern import Skyvern` → `skyvern.library.skyvern.Skyvern`
- **This is what users should import**
- Extends `AsyncSkyvern` (the Fern-generated API client)
- Adds browser-launching capabilities: `launch_local_browser()`, `use_cloud_browser()`, `connect_to_browser_over_cdp()`
- `Skyvern.local()` class method for embedded/local mode (uses in-process Skyvern server)
- `run_task()` and `run_workflow()` are overridden to add polling behavior
- Defined in: `skyvern/library/skyvern.py:L29`

### 2. `from skyvern.client import Skyvern` → `skyvern.client.client.Skyvern`
- Fern-generated raw API client (sync version)
- No browser launching — pure HTTP client
- Used internally; users typically use the `library` version
- `AsyncSkyvern` (async counterpart) at `skyvern/client/client.py:L2597`

### Library Module Structure (`skyvern/library/`)
| File | Purpose |
|------|---------|
| `skyvern.py` | Main `Skyvern` class + `Skyvern.local()` |
| `skyvern_browser.py` | `SkyvernBrowser` — browser session wrapper |
| `skyvern_browser_page.py` | `SkyvernPage` — page interaction |
| `skyvern_browser_page_agent.py` | AI agent on top of page |
| `skyvern_browser_page_ai.py` | AI-specific page methods |
| `skyvern_locator.py` | Element locator with AI fallback |
| `ai_locator.py` | AI-powered element locator |
| `embedded_server_factory.py` | Spins up in-process Skyvern server for `Skyvern.local()` |

### Script Generation System (`skyvern/core/script_generations/`)
The script system generates Playwright Python scripts from task runs for caching/speedup. On subsequent runs, the cached script executes first; on failure, AI fallback takes over.
- Entry: `skyvern/core/script_generations/generate_script.py`
- Execution: `skyvern/core/script_generations/skyvern_page_ai.py`
- API: `skyvern/core/script_generations/skyvern_page.py` (the `SkyvernPage` exposed to users)

### `skyvern/services/` Directory
Contains service wrappers for script blocks (functions like `run_task`, `extract`, `login`, `validate` etc. are importable directly from `skyvern`). These are used in Code Blocks within workflows.
