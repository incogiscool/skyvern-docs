# Repository Exploration Summary

## Project: Skyvern — AI-Powered Browser Automation

**Exploration Date:** March 19, 2025  
**Repository:** incogiscool/skyvern (Skyvern core repository)  
**Size:** ~45K Python LOC, ~13 core modules, 14 pages of documentation

---

## What Skyvern Is

Skyvern automates browser-based workflows using:
- **Vision LLMs** (OpenAI GPT-4o, Anthropic Claude, Azure, Gemini, etc. via LiteLLM)
- **Browser Automation** (Playwright-based)
- **Intelligent Agent Loop** (screenshot → LLM analysis → action planning → execution → repeat)

Instead of brittle XPath-based selectors, Skyvern reasons about page content and performs actions like a human would.

---

## Repository Structure

### Main Components

| Component | Language | Purpose |
|-----------|----------|---------|
| `skyvern/` | Python 3.11+ | Backend API, agent loop, browser orchestration |
| `skyvern-frontend/` | React 18 + TypeScript | Web UI for task creation, workflow editing, runs |
| `skyvern-ts/client/` | TypeScript | Published npm package (@skyvern/client) |
| `tests/` | Python + TypeScript | Unit, SDK, and smoke tests |
| `alembic/` | SQL | Database migrations |
| `docker-compose.yml` | Docker | Local dev stack (PostgreSQL, Redis, backend, frontend) |

### Backend Architecture (skyvern/)

```
FastAPI Application (api_app.py, forge/)
    ↓
Routes Layer (forge/sdk/routes/)
  - agent_protocol.py: Tasks, workflows, runs (3600+ lines)
  - scripts.py: Script CRUD and execution
  - credentials.py: Secret management (2100+ lines)
  - browser_sessions.py: Browser session lifecycle
  - streaming/: WebSocket/SSE handlers for real-time monitoring
    ↓
Services Layer (services/)
  - task_v2_service.py: Task execution orchestration
  - script_service.py: Script generation from goals
  - script_reviewer.py: Script quality validation
  - workflow_service.py: Workflow execution
  - block_service.py: Individual workflow block execution
  - webhook_service.py: Outbound webhooks
  - [other services: cleanup, otp, error_detection, pdf_import]
    ↓
Core Layer
  - agent.py: LLM agent loop (screenshot → analysis → action → repeat)
  - webeye/: Browser automation
    - browser_manager.py: Playwright lifecycle management
    - browser_state.py: State tracking
    - actions/: 50+ action types (click, input, scroll, navigate, extract_text, etc.)
  - forge/sdk/api/llm/: Multi-LLM provider routing via LiteLLM
    ↓
Data Layer
  - forge/sdk/db/models.py: SQLAlchemy ORM (30+ models)
  - PostgreSQL: Primary data store
  - S3/Azure Blob: Artifact storage (screenshots, recordings, logs)
  - Redis: Locks, caching, session data
```

### Key Databases

**SQLAlchemy Models (ORM):**
- TaskModel: Individual task execution
- WorkflowModel: Workflow definitions
- WorkflowRunModel: Workflow execution instances
- ScriptModel: Generated/user scripts
- StepModel: Task execution steps
- ActionModel: Individual browser actions
- ArtifactModel: Execution outputs (screenshots, logs, etc.)
- CredentialModel: Encrypted secrets
- BrowserSessionModel: Persistent browser state
- OrganizationModel: Multi-tenancy

**Schema (Pydantic):**
- TaskRequest, TaskResponse
- WorkflowRunRequest, WorkflowRunResponse
- ScriptRequest, ScriptResponse
- Various enums (TaskStatus, ArtifactType, BlockType, etc.)

### Frontend Architecture (React)

**Tech Stack:**
- Vite (build), TypeScript, TailwindCSS
- React Router (routing)
- TanStack Query (server state caching)
- Zustand (client state)
- Radix UI (accessible components)
- React Hook Form (form state)
- React Flow (workflow editor visualization)

**Routes:**
- `/tasks` — Task creation, listing, detail, running
- `/workflows` — Workflow editor, list, run history
- `/scripts` — Script management
- `/credentials` — Secret/credential management
- `/browser-sessions` — Session listing and monitoring
- `/settings` — Organization settings

### External Integrations

| Integration | Location | Purpose |
|-------------|----------|---------|
| **Langchain** | `integrations/langchain/` | LLM framework integration |
| **Llama Index** | `integrations/llama_index/` | RAG integration |
| **Make** | `integrations/make/` | Automation platform integration |
| **MCP** | `integrations/mcp/` + `cli/mcp.py` | Model Context Protocol support |

---

## Key Concepts

### Execution Models

1. **Tasks**
   - Single unit of work (navigate URL, fill form, extract data)
   - Execution: Agent loop with screenshot → LLM analysis → action
   - Statuses: created, queued, running, completed, failed, terminated, timed_out, canceled, skipped, paused

2. **Workflows**
   - DAG of blocks (Tasks, Scripts, Actions, Conditions, Loops, Code, Prompts, Webhooks)
   - Definition format: YAML or JSON
   - Features: Conditional branching, Jinja2 templating for parameter passing
   - Execution: Sequential block-by-block with output chaining

3. **Scripts**
   - Python or TypeScript code generated from task description
   - Generation: `script_service.py` uses LLM to write code
   - Review: `script_reviewer.py` validates quality/correctness
   - Usage: Standalone execution or as workflow block
   - Performance: Results cached for quick replay

4. **Browser Sessions**
   - Temporary: Task-scoped, discarded after completion
   - Persistent: Saved/reloadable for multi-step workflows
   - Storage: Session state in S3/Azure, screenshots for debugging
   - Streaming: VNC/CDP for real-time monitoring

### Data Concepts

**Artifacts:**
- Execution outputs: screenshots, recordings, logs, extracted data
- Storage: S3/Azure Blob (configurable)
- Access: Signed URLs for temporary public access

**Credentials:**
- Types: API keys, passwords, OAuth tokens, OTP/TOTP
- Storage: Encrypted in database (Fernet cipher)
- Integration: Optional vault sync (OnePassword, Bitwarden, Azure Key Vault)

**Actions:**
- Low-level browser interactions (click, input, scroll, navigate, extract_text, etc.)
- 50+ action types defined in `webeye/actions/action_types.py`
- Execution: Playwright-based via `webeye/actions/handler.py`
- Optimization: Previous actions replayable for performance

---

## API Surface (14 Main Endpoints)

### Tasks & Execution
- `POST /run/tasks` — Create and execute task
- `GET /runs/{run_id}` — Get execution details
- `GET /runs/{run_id}/artifacts` — Get execution outputs

### Workflows
- `POST /workflows` — Create workflow (YAML/JSON)
- `PUT /workflows/{workflow_id}` — Update definition
- `GET /workflows` — List workflows (with filtering)
- `POST /workflows/{workflow_id}/run` — Execute workflow

### Scripts
- `POST /scripts` — Create script
- `GET /scripts/{script_id}` — Get script details
- `POST /scripts/{script_id}/run` — Execute script

### Secrets & Configuration
- `POST /credentials` — Create credential
- `GET /credentials` — List credentials
- `DELETE /credentials/{credential_id}` — Delete credential

### Browser Management
- `POST /browser-sessions` — Create session
- WebSocket `/browser-sessions/{session_id}/stream` — Real-time monitoring

### Auxiliary
- `POST /run-blocks` — Execute single workflow block
- `POST /event_source` — Subscribe to execution events (SSE)

All routes behind JWT authentication, org-level authorization.

---

## Configuration (Key Variables)

**LLM Configuration:**
- `LLM_KEY`: Active model (default: OPENAI_GPT4O)
- `ENABLE_OPENAI`, `ENABLE_ANTHROPIC`, `ENABLE_AZURE`, `ENABLE_GEMINI`: Provider toggles
- Provider-specific keys and endpoints

**Database:**
- `DATABASE_STRING`: PostgreSQL connection
- `DATABASE_POOL_SIZE`: Connection pooling (default: 5)
- `REDIS_URL`: Redis connection for locks/cache

**Storage:**
- `SKYVERN_STORAGE_TYPE`: local | s3cloud | azureblob
- `AWS_S3_BUCKET_*`: S3 bucket names
- `ARTIFACT_STORAGE_PATH`: Local artifact path

**Browser:**
- `BROWSER_TYPE`: chromium-headful | firefox | webkit
- `BROWSER_WIDTH`, `BROWSER_HEIGHT`: Viewport (default: 1920x1080)
- `BROWSER_ACTION_TIMEOUT_MS`: Action timeout (default: 5000)
- `MAX_STEPS_PER_RUN`: Max task steps (default: 10)

**Logging:**
- `LOG_LEVEL`: DEBUG | INFO | WARNING | ERROR
- `JSON_LOGGING`: Structured JSON output (false by default)
- `DEBUG_MODE`: Verbose debugging (false by default)

Full config: `config.py` (800+ lines), `.env.example` (200+ lines)

---

## Dependencies

**Core Python:**
- FastAPI, Uvicorn (web framework)
- Pydantic (data validation)
- SQLAlchemy, Alembic, Psycopg (database)
- Playwright (browser automation)
- LiteLLM (multi-LLM provider)
- Anthropic, OpenAI (LLM SDKs)
- Structlog (structured logging)
- Jinja2 (templating)

**Storage & External:**
- aioboto3 (async AWS SDK)
- azure-storage-blob (Azure storage)
- Redis (distributed locks, caching)

**Frontend:**
- React, React Router, TanStack Query
- Vite, TypeScript, TailwindCSS
- Radix UI, CodeMirror, React Flow

---

## Development Workflow

### Local Setup
```bash
pip install skyvern           # Install Python SDK
skyvern quickstart            # Start server + UI
# Or manually:
docker-compose up             # PostgreSQL, Redis, backend, frontend
python -m alembic upgrade head # Database migrations
```

### Testing
- `pytest` — Unit tests
- `mypy skyvern/` — Type checking
- `tests/unit/`, `tests/sdk/`, `tests/smoke_tests/` directories

### Frontend Dev
- `npm run dev` — Dev server (Vite hot reload)
- `npm run build` — Production build
- `npm run lint` — ESLint

---

## Notable Patterns

### Async-First
- All I/O async (database, HTTP, files)
- AsyncSession for DB, aiohttp for HTTP, aiofiles for I/O
- No blocking operations in async context

### Lazy-Loaded IDs
- Each model has generator function (generate_task_id(), etc.)
- IDs include type prefix and UUID/timestamp

### Middleware Chain
- CORS, auth, error handling, request logging
- Custom exception handling for validation errors

### Structured Logging
- Structlog throughout (not print())
- `LOG = structlog.get_logger()`

### Jinja2 Templating
- Workflow blocks use `{{ }}` for variables
- Block outputs available downstream as `{block_label}_output`
- Strict validation of identifiers

---

## What's Missing / Unclear (25 Open Questions)

Critical areas needing clarification:
1. RBAC (role-based access control) — currently all org members have full access
2. Database replication (DATABASE_REPLICA_STRING exists but unused?)
3. Workflow scheduling/triggering — only manual execution visible
4. Long-running state persistence — how are server restarts handled?
5. Concurrency limits — max tasks per org, browser resource management
6. Artifact retention — cleanup policy unclear
7. WebSocket auth — same JWT as API or separate tokens?
8. Extension system — how are browser extensions loaded/used?
9. API versioning — v1 vs v2 vs legacy routes unclear
10. Cloud vs self-hosted — what features are cloud-only?

See `open-questions.md` for complete list.

---

## Documentation Produced

### 1. context-index.md (517 lines, 21KB)
Comprehensive reference with:
- Project overview & tech stack
- File tree and directory structure
- Layered architecture diagram
- Key concepts (tasks, workflows, scripts, etc.)
- Complete API surface (14+ endpoints)
- Data model (30+ models)
- Auth & middleware flow
- Configuration table (25+ variables)
- Dependencies (50+ libraries)
- Build & run instructions
- Patterns & conventions
- 10 open questions

**Purpose:** Single-source reference for all later documentation writers

### 2. open-questions.md (189 lines, 7KB)
25 unresolved questions organized by category:
- Architecture & Design (5 questions)
- Data & Persistence (3 questions)
- Authentication & Security (3 questions)
- Integration & Extensions (4 questions)
- Organization & Project Structure (3 questions)
- Performance & Observability (2 questions)
- Testing & Quality (2 questions)
- Deployment & Operations (3 questions)

**Purpose:** Track what needs deeper investigation before writing certain docs

### 3. page-proposal.md (136 lines, 6.1KB)
14-page documentation structure (APPROVED):
1. Getting Started (Guide)
2. Architecture (Explanation)
3. API Reference (Reference)
4. Workflows (Guide)
5. Scripts (Guide)
6. Browser Automation (Explanation)
7. Credentials & Auth (Guide)
8. Python SDK (Reference)
9. TypeScript SDK (Reference)
10. Deployment (Guide)
11. Monitoring & Observability (Guide)
12. Troubleshooting (Reference)
13. Integrations (Explanation)
14. Security (Explanation)

---

## Exploration Statistics

- **Time:** ~1 hour
- **Commands executed:** 40+
- **Files examined:** 100+
- **Key files read:** pyproject.toml, config.py, models.py, routes, schemas
- **Grep searches:** Pattern matching for routes, exports, types
- **Depth:** 3-4 levels of directory traversal

**Coverage:**
- ✅ Backend Python architecture (complete)
- ✅ Frontend React architecture (surveyed)
- ✅ API surface (documented)
- ✅ Database models (documented)
- ✅ Configuration (tabulated)
- ✅ Dependencies (listed)
- ✅ Integration points (noted)
- ⚠️ Test suite (noted but not analyzed in depth)
- ⚠️ CI/CD pipeline (not explored)
- ⚠️ Cloud deployment (not explored)

---

## Next Steps (For Documentation Writers)

1. **Read context-index.md first** — Single source of truth
2. **Consult open-questions.md** — Know what's unclear
3. **Follow page-proposal.md structure** — 14 pages approved
4. **Use file paths and line numbers** — All provided in index
5. **For API details** — Reference agent_protocol.py, scripts.py, credentials.py
6. **For data model** — Reference forge/sdk/db/models.py, schemas/
7. **For frontend** — Reference src/routes/, src/components/
8. **For deployment** — Reference docker-compose.yml, Dockerfile, config.py

**Writing guidance:**
- Keep language clear for mixed audience (technical + non-technical)
- Use code examples from actual codebase
- Reference file paths for readers who want to dive deeper
- Include diagrams for complex flows (architecture, authentication, execution)
- Link between pages (cross-references)

