# Exploration Summary

## What Was Explored

This document summarizes the exploration of the **Skyvern** codebase — an open-source browser automation platform using LLMs and computer vision.

**Repository**: `context/skyvern/` (imported from https://github.com/skyvern-ai/skyvern)

**Total codebase size**: ~13 subdirectories, hundreds of Python files, React frontend, TypeScript SDK, database migrations

---

## Key Findings

### Project Overview
- **Type**: Full-stack web application (Python backend + React frontend + TypeScript SDK)
- **Purpose**: Automate browser-based workflows using AI vision and LLMs
- **Architecture**: Task-driven agent model inspired by BabyAGI/AutoGPT with Playwright integration
- **Scale**: Multi-tenant, cloud-ready (Skyvern Cloud offering), supports local deployment

### Technology Stack
- **Backend**: Python 3.11+, FastAPI, SQLAlchemy, asyncpg, Playwright
- **LLM Support**: OpenAI, Anthropic (Claude), Google Vertex AI, Azure, via LiteLLM abstraction
- **Database**: PostgreSQL with Alembic migrations
- **Frontend**: React 18+, TypeScript, Vite, Tailwind CSS
- **Deployment**: Docker, Docker Compose, Kubernetes-ready

### Main Components

1. **Agent** (`forge/agent.py`): Orchestrates task execution, calls LLMs, executes actions
2. **API** (`forge/api_app.py`): FastAPI server with routes for tasks, workflows, credentials, browser sessions
3. **Database** (`forge/sdk/db/`): SQLAlchemy ORM models (Task, Step, Workflow, BrowserSession, etc.)
4. **Library** (`library/`): High-level Python SDK (Skyvern, SkyvernBrowser, SkyvernBrowserPage)
5. **Services** (`services/`): Business logic (action execution, browser management, webhooks, workflows)
6. **Frontend** (`skyvern-frontend/`): React UI for task/workflow management
7. **Workflow Engine** (`forge/sdk/workflow/`): DAG-based workflow orchestration with block types

### Key Architectural Patterns

- **Lazy imports**: Public API uses deferred imports to reduce startup time
- **Async-first**: All I/O operations are async (database, HTTP, browser)
- **Structured logging**: Contextual logging with request IDs, org IDs via structlog
- **Request-scoped context**: SkyvernContext for managing per-request data
- **Multi-tenancy**: organization_id field throughout schema, tenant isolation at API level

---

## Documentation Deliverables

### 1. `context/context-index.md` (836 lines, 36KB)

**Purpose**: Comprehensive reference for all later documentation writers

**Sections**:
- Overview and stack
- File tree (annotated with descriptions)
- System architecture (data flow diagrams)
- Key concepts (Task, Step, Workflow, Block, Artifact, etc.)
- API surface (main endpoints and schemas)
- Data model (database models and relationships)
- Auth and middleware
- Configuration table (key environment variables)
- Dependencies (key libraries and their roles)
- Build and run instructions
- Patterns and conventions
- Open questions

### 2. `context/open-questions.md` (106 lines, 6.5KB)

**Purpose**: Track unknowns and areas requiring deeper investigation

**Categories**:
- Architecture & design (8 questions)
- Authentication & multi-tenancy (3 questions)
- LLM & AI (4 questions)
- Database & persistence (3 questions)
- Frontend & state management (3 questions)
- CLI & local development (2 questions)
- Deployment & operations (3 questions)
- Edge cases & error handling (3 questions)
- Type system & validation (2 questions)
- Testing & quality (2 questions)
- Documentation & examples (2 questions)

**Investigation priority**: High/Medium/Low classifications for each question

### 3. `context/page-proposal.md` (32 lines, 2.8KB)

**Purpose**: Approved documentation structure

**13 proposed pages**:
- 8 Guides (getting-started, sdk-guide, workflows, browser-automation, credentials-and-auth, data-extraction, error-handling, deployment)
- 2 Explanations (architecture, agents-and-ai)
- 3 References (api-reference, cli-reference, examples)

**Status**: ✅ Developer approved

---

## How to Use These Documents

### For Documentation Writers

1. **Start with `context-index.md`**: Read the Overview, Architecture, and Key Concepts sections to understand the system
2. **Deep dive by page**: For each assigned page, find relevant sections in the index (e.g., for API docs, see "API Surface" section)
3. **Check `open-questions.md`**: If something is unclear, check if it's listed with investigation priority
4. **Reference code**: File paths in the index point to exact locations in `context/skyvern/` for verification

### For Developers Contributing Code

1. **Patterns & Conventions**: See the corresponding section in `context-index.md` for how this project structures code
2. **Database Changes**: Review the "Data Model" section and check `forge/sdk/db/models.py` for similar patterns
3. **Adding Endpoints**: See "API Surface" section for existing patterns and "routes/" directory structure
4. **Testing**: See "Patterns & Conventions" → Testing subsection

### For Project Maintainers

1. **Architecture reference**: Use "Architecture" section for design reviews
2. **Unknown areas**: Check `open-questions.md` for critical unknowns that should be clarified
3. **Configuration audit**: Use the Configuration table to ensure all important settings are documented

---

## What's Not Included

- **Detailed code walkthroughs**: Line-by-line analysis of specific files
- **Git history analysis**: Only a brief note that this is a documentation project
- **Performance benchmarks**: No load test results or latency data
- **Competitive analysis**: No comparison with other automation tools
- **Business/product details**: Focus is on technical architecture

---

## Next Steps

1. **Assign pages**: Distribute the 13 pages from `page-proposal.md` to documentation writers
2. **Resolve open questions**: Have architects/lead devs answer high-priority questions from `open-questions.md`
3. **Write content**: Use `context-index.md` as the reference source for all pages
4. **Add examples**: Create code examples for common use cases (form filling, data extraction, etc.)
5. **Cross-reference**: Link between pages (e.g., api-reference ↔ sdk-guide ↔ examples)
6. **Maintain**: Keep the index updated as the codebase evolves

---

## Files Created

```
context/
├── context-index.md          # 836 lines - Comprehensive technical reference
├── open-questions.md         # 106 lines - Unresolved questions and unknowns
├── page-proposal.md          # 32 lines - Approved documentation structure
└── EXPLORATION_SUMMARY.md    # This file - Overview of the exploration process
```

---

## Statistics

| Metric | Value |
|--------|-------|
| Files explored | 300+ (Python, JS/TS, YAML, SQL) |
| Directories surveyed | 13 main directories |
| Code modules documented | ~40 modules |
| Database models listed | 7 core models |
| API endpoints categorized | 6 major route groups |
| Configuration variables | 30+ documented |
| Open questions identified | 32 questions across 11 categories |
| Documentation pages proposed | 13 pages (8 guides, 2 explanations, 3 references) |

---

**Exploration completed**: 2026-03-18
**Total documentation created**: ~975 lines across 3 markdown files

