# START HERE — Repository Context for Documentation Writers

Welcome! This directory contains everything you need to write documentation for the Skyvern project.

## 🚀 Quick Start (5 minutes)

1. **Read this file** ← You are here
2. **Read `context/README.md`** ← Orientation guide
3. **Read `context/context-index.md`** ← Complete technical reference (mandatory)
4. **Skim `context/EXPLORATION_SUMMARY.md`** ← Big picture overview
5. **Check `context/open-questions.md`** ← Before writing uncertain sections
6. **Follow `context/page-proposal.md`** ← Documentation structure

---

## 📁 What's in This Directory?

| File | Purpose | Read Time |
|------|---------|-----------|
| **context-index.md** | Complete technical architecture with file paths, APIs, models, config | 30 min |
| **open-questions.md** | 25 unresolved questions (check before writing sensitive sections) | 10 min |
| **page-proposal.md** | 14 approved documentation pages with scope/audience | 5 min |
| **EXPLORATION_SUMMARY.md** | Detailed findings, statistics, next steps | 15 min |
| **README.md** | How to use these context files | 5 min |
| **START_HERE.md** | This file | 2 min |

**Total recommended reading time: ~1 hour**

---

## 🎯 What You Need to Know

### The Project
**Skyvern** is an AI-powered browser automation platform. Instead of brittle XPath selectors, it uses:
- **Vision LLMs** (OpenAI, Anthropic, Azure, Google, etc.)
- **Browser Automation** (Playwright)
- **Intelligent Agent Loop** (screenshot → analysis → action planning → execution → repeat)

### The Stack
- **Backend:** Python 3.11+, FastAPI, SQLAlchemy, Playwright
- **Frontend:** React 18, TypeScript, TailwindCSS
- **Database:** PostgreSQL with Alembic migrations
- **Storage:** S3/Azure Blob for artifacts
- **Cache/Locks:** Redis

### Key Concepts
- **Tasks:** Single automation unit (navigate, fill form, extract data)
- **Workflows:** DAG of blocks with templating and branching
- **Scripts:** Generated Python/TypeScript code
- **Browser Sessions:** Persistent or temporary automation state
- **Credentials:** Encrypted secrets (API keys, passwords, OTP)
- **Artifacts:** Execution outputs (screenshots, logs, data)

---

## 📖 Documentation Structure (14 Pages)

Your documentation will be organized in 14 pages (approved by developer):

### Guides (5 pages)
1. **Getting Started** — Installation, first task, setup
2. **Workflows** — Structure, blocks, templating, execution
3. **Scripts** — Generation, actions, deployment
4. **Credentials & Auth** — Management, types, security
5. **Deployment** — Local, Docker, Kubernetes

### Explanations (5 pages)
1. **Architecture** — System design, layers, execution flow
2. **Browser Automation** — How it works, sessions, proxy
3. **Integrations** — Langchain, Llama Index, Make, MCP
4. **Security** — Authentication, encryption, best practices

### References (4 pages)
1. **API Reference** — Endpoint documentation
2. **Python SDK** — SDK API surface
3. **TypeScript SDK** — NPM package API
4. **Troubleshooting** — Common issues, solutions

---

## 🔑 Key Information at a Glance

### Backend Architecture
```
FastAPI (api_app.py)
  ↓
Routes (forge/sdk/routes/)
  • agent_protocol.py — Tasks, workflows, runs (3600+ lines)
  • scripts.py — Script CRUD
  • credentials.py — Secrets (2100+ lines)
  ↓
Services (services/)
  • task_v2_service.py — Task orchestration
  • script_service.py — Generation + review
  • workflow_service.py — Workflow execution
  ↓
Core (webeye/, agent.py)
  • Playwright browser automation
  • LLM agent loop (multi-provider)
  ↓
Data (SQLAlchemy, PostgreSQL, S3/Azure)
```

### Main API Endpoints (14+)
- `POST /run/tasks` — Execute task
- `POST /workflows` — Create workflow
- `POST /workflows/{id}/run` — Execute workflow
- `POST /scripts` — Create/generate script
- `POST /credentials` — Manage secrets
- `POST /browser-sessions` — Manage browser state
- WebSocket `/browser-sessions/{id}/stream` — Real-time monitoring

All endpoints are JWT-authenticated, organization-scoped.

### Database Models (30+)
Key ones: TaskModel, WorkflowModel, WorkflowRunModel, ScriptModel, StepModel, ActionModel, ArtifactModel, CredentialModel, BrowserSessionModel, OrganizationModel

---

## 🛠 Before You Start Writing

### Checklist
- [ ] Read `context-index.md` completely (most important!)
- [ ] Review `page-proposal.md` to see your page's scope/audience
- [ ] Check `open-questions.md` if your page covers unclear areas
- [ ] Note file paths from `context-index.md` for code references
- [ ] Understand the audience for your page (technical vs business users)

### During Writing
- Use actual file paths from `context-index.md` when referencing code
- Include code examples from the repository
- Create ASCII diagrams for complex flows
- Cross-reference between pages using the structure in `page-proposal.md`
- If you find something unclear, check `open-questions.md`

### If You Get Stuck
1. **Check context-index.md** — It has file paths and detailed info
2. **Check EXPLORATION_SUMMARY.md** — It has guidance and big-picture context
3. **Check open-questions.md** — It might be a known unclear area
4. **Ask the developer** — Use the ask_developer() tool

---

## 📊 By the Numbers

- **~45,000 lines** of Python backend code
- **30+ database models** with relationships
- **14+ main API endpoints** (plus streaming)
- **50+ external dependencies**
- **6+ frontend page groups**
- **25 open questions** that need clarification
- **14 documentation pages** to write

---

## 🎓 For Each Page You Write

1. **Read the scope** in `page-proposal.md`
2. **Understand the audience** (technical vs non-technical)
3. **Check if it's unclear** in `open-questions.md`
4. **Reference actual code** using file paths from `context-index.md`
5. **Include examples** from the codebase
6. **Link to related pages** from other documentation

---

## 📞 Questions?

If something in the context files is unclear:
1. Check the README.md for file conventions
2. Re-read the specific section in context-index.md
3. See if it's documented in open-questions.md
4. Ask the developer using ask_developer()

---

## ✅ You're Ready!

You have everything needed to write excellent Skyvern documentation:
- ✅ Complete technical reference (context-index.md)
- ✅ Known unclear areas (open-questions.md)
- ✅ Approved page structure (page-proposal.md)
- ✅ Big-picture guidance (EXPLORATION_SUMMARY.md)
- ✅ File conventions (README.md)

**Next step:** Read `context-index.md` for complete technical details.

---

**Last updated:** March 19, 2025  
**Repository:** Skyvern (AI-Powered Browser Automation)  
**Status:** ✅ Ready for documentation writers

