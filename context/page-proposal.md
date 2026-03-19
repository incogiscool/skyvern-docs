# Documentation Page Proposal — APPROVED

## Pages List

1. Getting Started — Guide — **Outcome:** Users will have Skyvern installed and running locally, with their first task executed successfully.
2. Architecture — Explanation — **Outcome:** Readers will understand how Skyvern's layered architecture (API → services → core) works and how components interact.
3. API Reference — Reference — **Outcome:** Developers will have complete endpoint documentation with request/response schemas for all Skyvern APIs.
4. Workflows — Guide/Explanation — **Outcome:** Users will be able to design, structure, and deploy multi-step automation workflows with conditional logic and templating.
5. Scripts — Guide/Explanation — **Outcome:** Developers will understand how to generate, review, deploy, and optimize scripts for reusable automation.
6. Browser Automation — Explanation — **Outcome:** Users will understand how Skyvern automates browsers, manages sessions, and handles proxy/geolocation configuration.
7. Credentials & Auth — Guide — **Outcome:** Users will know how to create, manage, and securely use credentials in tasks and workflows.
8. Python SDK — Reference — **Outcome:** Python developers will have complete API documentation and examples for the Skyvern Python SDK.
9. TypeScript SDK — Reference — **Outcome:** TypeScript/JavaScript developers will have complete API documentation and examples for the @skyvern/client npm package.
10. Deployment — Guide — **Outcome:** DevOps teams will be able to deploy Skyvern locally, in Docker, and on Kubernetes with proper configuration.
11. Monitoring & Observability — Guide — **Outcome:** Operators will know how to configure logging, monitor performance, and debug task executions.
12. Troubleshooting — Reference — **Outcome:** Users will have solutions to common issues (crashes, timeouts, rate limits) and tools for diagnosis.
13. Integrations — Explanation — **Outcome:** Users will understand how to integrate Skyvern with Langchain, Llama Index, Make, and MCP.
14. Security — Explanation — **Outcome:** Teams will understand Skyvern's authentication, encryption, and best practices for secure deployment.

---

## Proposed Page Structure

### 1. Getting Started — Guide
Walk new developers through local setup (Python SDK install, quickstart command, Docker Compose), first task creation via API, understanding the task execution flow.

**Scope:** Installation, first run, basic concepts introduction
**Audience:** New users, developers trying the platform
**Key sections:** Requirements, install methods, quickstart, first task, next steps

---

### 2. Architecture — Explanation
System design (Vision LLM + browser automation loop), layered architecture (API → services → core), key abstractions (tasks, workflows, scripts, blocks), data model overview.

**Scope:** How the system works, design decisions, high-level concepts
**Audience:** Technical leads, developers wanting deep understanding
**Key sections:** System overview, components, data flow, execution models

---

### 3. API Reference — Reference
Complete API endpoint documentation (tasks, workflows, scripts, credentials, browser sessions, streaming), request/response schemas, error codes.

**Scope:** All public API endpoints, request/response formats, error handling
**Audience:** Backend developers, API integrators
**Key sections:** Endpoint listing, request formats, response examples, error codes, authentication

---

### 4. Workflows — Guide/Explanation
What are workflows, how to structure them (blocks, branching, templating), parameter passing between blocks, conditional logic, how to test and deploy workflows.

**Scope:** Workflow concepts, block types, YAML definition, templating, execution
**Audience:** Users building automations, business analysts
**Key sections:** Workflow structure, block types, parameters, templating (Jinja2), testing, deployment, examples

---

### 5. Scripts — Guide/Explanation
Script generation from goals, available actions (click, input, navigate, extract), how scripts differ from workflows, when to use scripts vs workflows, script caching and deployment.

**Scope:** Script-based automation, action types, generation, deployment
**Audience:** Developers, technical users
**Key sections:** Script generation, action reference, SDK usage, deployment, performance (caching)

---

### 6. Browser Automation — Explanation
How Skyvern automates browsers (Playwright under the hood), browser sessions (temporary vs persistent), proxy/geolocation configuration, handling anti-bot detection.

**Scope:** Browser mechanics, session management, configuration
**Audience:** Advanced users, operators
**Key sections:** How automation works, browser sessions, proxy/geo configuration, anti-bot handling, browser profiles

---

### 7. Credentials & Auth — Guide
Credential types supported (API keys, passwords, OTP, vault integration), how to create and use credentials in tasks/workflows, security best practices.

**Scope:** Credential management, types, usage in automations, security
**Audience:** Users building automations, security-conscious teams
**Key sections:** Credential types, creation/management, usage in workflows/scripts, vault integration, security best practices

---

### 8. Python SDK — Reference
Python SDK API (Skyvern class, decorator methods), example scripts, SDK vs API differences.

**Scope:** Python SDK API surface, usage examples, capabilities
**Audience:** Python developers
**Key sections:** Installation, client setup, available methods, decorators (@action, @cached, @workflow), examples, SDK vs API differences

---

### 9. TypeScript SDK — Reference
TypeScript SDK API (@skyvern/client), browser-compatible usage, example code.

**Scope:** TypeScript SDK API surface, usage examples, npm package
**Audience:** TypeScript/JavaScript developers
**Key sections:** Installation (@skyvern/client), client setup, available methods, examples, Node.js vs browser usage

---

### 10. Deployment — Guide
Local development (requirements, setup), Docker Compose for local dev, Docker for production, Kubernetes deployment, environment configuration, database setup (PostgreSQL, Alembic migrations).

**Scope:** Running Skyvern locally and in production
**Audience:** DevOps, operators, self-hosted users
**Key sections:** Local setup (requirements, installation), Docker Compose, Docker production images, Kubernetes, environment variables, database setup, scaling considerations

---

### 11. Monitoring & Observability — Guide
Logging configuration (JSON_LOGGING, LOG_LEVEL, structlog), performance monitoring (PostHog analytics), debugging task runs (artifacts, streaming video), error diagnosis.

**Scope:** Observability, logging, monitoring, debugging
**Audience:** Operators, DevOps, developers troubleshooting issues
**Key sections:** Logging setup, structured logs, analytics integration, monitoring metrics, debugging tools (artifacts, streaming, debug mode)

---

### 12. Troubleshooting — Reference
Common issues and solutions (browser crashes, timeout errors, LLM rate limits, database connection issues), debug mode (`DEBUG_MODE` config), artifact inspection.

**Scope:** Problem diagnosis and resolution
**Audience:** Users experiencing issues, operators
**Key sections:** Common issues with solutions, debug mode, artifact inspection, log analysis, support contacts

---

### 13. Integrations — Explanation
Available integrations (Langchain, Llama Index, Make, MCP), how to use them, examples.

**Scope:** Third-party integrations and extensions
**Audience:** Users wanting to integrate with other tools
**Key sections:** Integration overview, Langchain integration, Llama Index integration, Make integration, MCP integration, examples

---

### 14. Security — Explanation
Authentication & authorization (JWT, org-level access), encryption (credentials, artifacts), security best practices, CORS configuration.

**Scope:** Security architecture, practices, configuration
**Audience:** Security teams, operators, users with compliance needs
**Key sections:** Authentication flow, authorization model, encryption, secrets management, CORS, deployment security, compliance considerations

---

## Status

✅ **APPROVED** by developer

All 14 pages approved as proposed. Ready to begin writing documentation.

