# Documentation Context Files

This directory contains the exploration results and planning documents for the Skyvern documentation project.

## Files

### 1. **context-index.md** (21 KB, 517 lines)
**PRIMARY REFERENCE DOCUMENT**

The authoritative reference for all aspects of the Skyvern codebase. This is the document to read first.

**Contents:**
- Project overview and tech stack
- Complete file tree and directory structure
- Layered architecture diagrams
- Key concepts (tasks, workflows, scripts, etc.)
- Complete API surface documentation
- Data model (30+ database models)
- Authentication and middleware flow
- Configuration reference (25+ variables)
- Dependencies (50+ libraries)
- Build, test, and run instructions
- Coding patterns and conventions
- 10 critical open questions

**For Documentation Writers:** Use this as your primary reference. All file paths, line numbers, and structural information are documented here.

---

### 2. **open-questions.md** (7 KB, 189 lines)
**INVESTIGATION CHECKLIST**

25 unresolved questions grouped by category that need deeper investigation before finalizing certain documentation sections.

**Categories:**
- Architecture & Design (5 questions)
- Data & Persistence (3 questions)
- Authentication & Security (3 questions)
- Integration & Extensions (4 questions)
- Organization & Project Structure (3 questions)
- Performance & Observability (2 questions)
- Testing & Quality (2 questions)
- Deployment & Operations (3 questions)

**For Documentation Writers:** Consult this before writing sections about RBAC, database replication, workflow scheduling, deployment architecture, or cloud vs self-hosted features. These areas need clarification.

---

### 3. **page-proposal.md** (6.1 KB, 136 lines)
**DOCUMENTATION STRUCTURE (APPROVED)**

14-page documentation structure approved by the developer. This defines the scope and audience for each page.

**Pages:**
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

**For Documentation Writers:** Follow this structure. Each page has defined scope, audience, and key sections.

---

### 4. **EXPLORATION_SUMMARY.md** (14 KB, ~450 lines)
**EXPEDITION REPORT**

Detailed summary of the repository exploration process, findings, and statistics.

**Contents:**
- What Skyvern is (overview and mission)
- Repository structure (all major components)
- Backend architecture (layered diagram)
- Database models and schemas
- Frontend architecture and tech stack
- External integrations
- Key execution concepts
- 14+ main API endpoints
- Configuration variables and their purposes
- Dependencies breakdown
- Development workflow
- Notable patterns and conventions
- 25 unclear areas/questions
- Documentation produced
- Exploration statistics
- Guidance for next steps

**For Documentation Writers:** Use this to understand the big picture and get guidance on which files to reference for specific topics.

---

## Quick Start for Documentation Writers

1. **Read context-index.md first** — Get the complete technical overview
2. **Consult open-questions.md** — Know what's unclear before writing
3. **Follow page-proposal.md** — Use the approved 14-page structure
4. **Reference EXPLORATION_SUMMARY.md** — For guidance and statistics

## Key Statistics

| Metric | Value |
|--------|-------|
| Python LOC (skyvern/) | ~45,000 |
| Database Models | 30+ |
| API Endpoints (main) | 14+ |
| Configuration Variables | 50+ |
| External Dependencies | 50+ |
| Frontend Routes | 6+ |
| Documentation Pages (proposed) | 14 |
| Open Questions | 25 |

## Exploration Scope

✅ **Complete:**
- Backend Python architecture
- API surface and endpoints
- Database models and relationships
- Frontend React architecture
- Configuration system
- Dependencies

⚠️ **Surveyed but not deep-dived:**
- Test suite details
- CI/CD pipeline
- Cloud-specific deployment
- Performance benchmarks

## Document Conventions

- **File paths** are relative to `context/skyvern/`
- **Line numbers** refer to actual source code (L10-L45 means lines 10-45)
- **Architecture diagrams** are ASCII art for easy reference
- **API endpoints** include file location and handler function name
- **Configuration variables** include default values and file location

## For Questions

If documentation writers encounter ambiguities:
1. Check open-questions.md to see if it's already identified
2. Search context-index.md for related file paths
3. Reference EXPLORATION_SUMMARY.md for context and guidance
4. Ask the developer (use ask_developer() tool)

