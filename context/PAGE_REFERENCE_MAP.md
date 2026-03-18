# Page Reference Map

**Quick lookup**: For each documentation page, which sections of context-index.md to read.

---

## ✅ Complete Pages

### 1. getting-started.md
**Status**: ✅ COMPLETE  
**Writer**: Manicule  
**Sections Used**:
- Overview (L1-30)
- Build & Run (L702-750)
- Configuration (L630-680) — for env vars

**Additional Source Files Referenced**:
- context/skyvern/.env.example
- context/skyvern/skyvern/cli/quickstart.py
- context/skyvern/docker-compose.yml

---

### 2. architecture.md
**Status**: ✅ COMPLETE  
**Writer**: Manicule  
**Sections Used**:
- Overview (L1-30)
- Architecture (L152-250)
- Key Concepts (L252-420)
- Data Model (L482-600) — briefly

**Additional Source Files Referenced**:
- context/skyvern/skyvern/forge/agent.py
- context/skyvern/skyvern/forge/api_app.py
- context/skyvern/skyvern/forge/sdk/workflow/service.py
- context/skyvern/skyvern/__init__.py

---

## 🟡 In Progress / To Do

### 3. api-reference.md
**Type**: Reference  
**Priority**: HIGH  
**Length Target**: 300-400 lines

**context-index.md Sections**:
- Overview (L1-30) — Quick summary
- API Surface (L422-480) — PRIMARY: All endpoints listed here
- Data Model (L482-600) — Request/response schemas
- Configuration (L630-680) — Auth headers, API key config

**Key Source Files to Reference**:
- `forge/sdk/routes/routers.py` — Router composition
- `forge/sdk/routes/tasks.py` — Task endpoints
- `forge/sdk/routes/workflows.py` — Workflow endpoints
- `forge/sdk/routes/browser_sessions.py` — Browser session endpoints
- `forge/sdk/routes/credentials.py` — Credential endpoints
- `forge/sdk/routes/browser_profiles.py` — Browser profile endpoints
- `forge/sdk/routes/debug_sessions.py` — Debug endpoints
- `forge/sdk/routes/agent_protocol.py` — Agent protocol
- `forge/sdk/schemas/tasks.py` — TaskRequest, TaskResponse models
- `forge/sdk/schemas/workflows.py` — WorkflowRequest models
- `exceptions.py` — Error codes and formats

**What to Include**:
1. Overview (1 paragraph)
2. Authentication (headers, API keys, JWT)
3. Task Endpoints (create, get, list, update status, cancel)
4. Workflow Endpoints (create, get, list, run, update)
5. Browser Session Endpoints (create, get, list, delete)
6. Credential Endpoints (create, get, list, delete)
7. Other Endpoints (profiles, debug, agent protocol)
8. Request/Response Examples for each
9. Error Codes & Status Codes
10. Rate Limiting (if applicable)

**Cross-References**:
- getting-started.md (for first API call example)
- sdk-guide.md (Python SDK wrapper around API)
- workflows.md (workflow definition format)

---

### 4. sdk-guide.md
**Type**: Guide  
**Priority**: HIGH  
**Length Target**: 400-500 lines

**context-index.md Sections**:
- Overview (L1-30) — Quick summary
- Key Concepts (L252-280) — Skyvern class, SkyvernPage, decorators
- Architecture (L200-220) — SDK layer overview
- Dependencies (L682-700) — External packages used

**Key Source Files to Reference**:
- `skyvern/__init__.py` — Exports (Skyvern, SkyvernPage, workflow, action, etc.)
- `skyvern/skyvern_context.py` — SkyvernContext for request context
- `skyvern/browser.py` — SkyvernBrowser class
- `skyvern/page.py` — SkyvernBrowserPage class
- `skyvern/workflow/__init__.py` — @workflow, @action decorators
- `skyvern/actions/actions.py` — Action classes (Click, Fill, etc.)
- Python SDK types/classes in skyvern/

**What to Include**:
1. Overview: What is the SDK? When to use it?
2. Installation & Setup
3. Basic Usage: Import Skyvern, create task
4. Skyvern Class: Methods, parameters, API key
5. SkyvernBrowser & SkyvernBrowserPage: Navigation, interaction
6. Decorators:
   - @workflow — Define workflows
   - @action — Define actions
   - @cached — Cache results
   - @login — Handle login
7. Actions: Click, Fill, Select, Scroll, etc.
8. Workflow DSL: Using decorators to define workflows
9. Error Handling: Common errors, debugging
10. TypeScript SDK: Brief intro and link
11. Advanced: Credentials, context, custom prompts

**Cross-References**:
- api-reference.md (for API calls underneath)
- workflows.md (for complex workflow examples)
- examples.md (for real-world use cases)

---

### 5. workflows.md
**Type**: Guide  
**Priority**: HIGH  
**Length Target**: 400-500 lines

**context-index.md Sections**:
- Key Concepts (L370-420) — Block types and execution
- Architecture (L220-250) — Workflow engine
- Data Model (L550-600) — WorkflowModel, block models

**Key Source Files to Reference**:
- `forge/sdk/workflow/models/` — All block types
- `forge/sdk/workflow/context_manager.py` — WorkflowRunContext
- `forge/sdk/workflow/service.py` — WorkflowService
- `forge/sdk/workflow/workflow_definition_converter.py` — Definition format

**What to Include**:
1. Overview: What are workflows?
2. Basic Structure: Steps, blocks, sequence
3. Block Types:
   - ActionBlock (browser actions)
   - ValidationBlock (assertions)
   - LoopBlock (repetition)
   - ConditionalBlock (branching)
   - ParallelBlock (concurrency)
   - Other blocks
4. Parameters & Variables:
   - How to define parameters
   - How to pass variables between blocks
   - Variable interpolation syntax
5. Nesting: How to nest blocks
6. Execution Flow: How blocks execute
7. Data Passing: How data flows between blocks
8. Error Handling: What happens on block failure
9. Examples: Simple to complex workflows
10. Syntax: YAML/JSON/Python format

**Cross-References**:
- sdk-guide.md (for @workflow decorator)
- examples.md (for realistic workflows)
- api-reference.md (for workflow creation API)

---

### 6. agents-and-ai.md
**Type**: Explanation  
**Priority**: MEDIUM  
**Length Target**: 350-400 lines

**context-index.md Sections**:
- Overview (L1-30) — Tech stack: LiteLLM, AI
- Architecture (L175-195) — Agent core
- Key Concepts (L310-350) — Step, LLM, vision
- Dependencies (L682-700) — LLM libraries

**Key Source Files to Reference**:
- `forge/agent.py` — Agent class, step generation, LLM calling
- `forge/agent_functions.py` — Agent helper functions
- `forge/prompts.py` — Prompt templates
- `forge/forge_openai_client.py` — OpenAI client wrapper
- `forge/sdk/api/llm/` — LLM provider support
- `forge/sdk/db/models.py` — StepModel (token counting)

**What to Include**:
1. Overview: What is an agent?
2. Agent Loop:
   - Perception: Screenshot analysis
   - Decision: LLM-based step generation
   - Action: Browser execution
3. LLM Providers:
   - OpenAI (GPT-4, GPT-4V)
   - Anthropic (Claude)
   - Google Vertex AI
   - How to configure each
4. Vision & Screenshot Analysis:
   - When vision LLMs are used
   - How screenshots are processed
   - Cost implications
5. Prompt Engineering:
   - How prompts are constructed
   - Available templates
   - Customization options
6. Token Counting & Cost:
   - How tokens are tracked
   - Cost optimization
   - Provider differences
7. Step Generation: How agent decides next step
8. Error Recovery: How agent handles failures
9. Configuration: LLM settings, timeouts, retries

**Cross-References**:
- architecture.md (for agent architecture)
- credentials-and-auth.md (for API key setup)
- deployment.md (for LLM env vars)

---

### 7. browser-automation.md
**Type**: Guide  
**Priority**: MEDIUM  
**Length Target**: 350-400 lines

**context-index.md Sections**:
- Key Concepts (L295-330) — Vision, browser control, locators
- Architecture (L200-220) — Browser layer
- Dependencies (L682-700) — Playwright

**Key Source Files to Reference**:
- `skyvern/browser.py` — Browser control
- `skyvern/page.py` — Page interaction
- `forge/sdk/api/browser/` — Browser abstraction
- `forge/agent.py` — Vision LLM calls for locator finding
- `skyvern/actions/actions.py` — Action implementations

**What to Include**:
1. Overview: Browser automation with AI
2. Browser Sessions:
   - Creating and managing sessions
   - Session persistence
   - Cleanup
3. Navigation:
   - goto_url()
   - refresh()
   - back()
4. Interaction:
   - click() — CSS vs AI-powered locators
   - fill() — Text input
   - select() — Dropdowns
   - hover(), scroll()
5. AI-Powered Locators:
   - How they work
   - When to use vs CSS selectors
   - Cost implications
6. Screenshot & Vision:
   - Taking screenshots
   - Vision LLM analysis
   - Element detection
7. Waiting & Loading:
   - Wait strategies
   - Handling dynamic content
   - Timeout configuration
8. Cookie & Storage Management
9. Accessibility: Using accessibility locators
10. Advanced: CDPs, debugging, VNC

**Cross-References**:
- getting-started.md (basic usage)
- agents-and-ai.md (vision LLM details)
- error-handling.md (debugging techniques)

---

### 8. credentials-and-auth.md
**Type**: Guide  
**Priority**: MEDIUM  
**Length Target**: 350-400 lines

**context-index.md Sections**:
- Architecture (L230-250) — Auth layer
- Data Model (L520-550) — CredentialModel, UserModel
- Configuration (L630-680) — Auth env vars
- Key Concepts (L360-380) — Auth mechanisms

**Key Source Files to Reference**:
- `forge/sdk/db/models.py` — CredentialModel, UserModel
- `forge/sdk/core/security.py` — JWT, webhook signatures
- `forge/sdk/routes/credentials.py` — Credential endpoints
- `skyvern/config.py` — JWT settings
- `forge/api_app.py` — Auth middleware

**What to Include**:
1. Overview: Auth in Skyvern
2. API Authentication:
   - API keys
   - JWT tokens
   - Bearer token format
3. Credentials Management:
   - Types of credentials (API keys, passwords, OAuth)
   - Creating credentials
   - Retrieving credentials
   - Deleting credentials
   - Encryption at rest
4. Multi-tenancy:
   - Organization isolation
   - User roles
   - Org context enforcement
5. JWT Tokens:
   - Token structure and claims
   - Token expiration
   - Refresh tokens
6. Webhook Security:
   - Webhook signature verification
   - Signing algorithm
   - Header format
7. Best Practices:
   - Never commit API keys
   - Using environment variables
   - Rotating credentials
8. Troubleshooting:
   - Auth errors
   - Token expiration
   - Permission errors

**Cross-References**:
- getting-started.md (API key setup)
- deployment.md (env vars)
- api-reference.md (API key headers)

---

### 9. data-extraction.md
**Type**: Guide  
**Priority**: MEDIUM  
**Length Target**: 350-400 lines

**context-index.md Sections**:
- Key Concepts (L310-325) — Extraction goal, schema
- Data Model (L560-580) — TaskModel extraction fields
- Architecture (L205-220) — Agent extraction logic

**Key Source Files to Reference**:
- `forge/sdk/schemas/tasks.py` — TaskRequest.extraction_goal
- `forge/agent.py` — Extraction logic in agent
- `forge/sdk/db/models.py` — TaskModel.extraction

**What to Include**:
1. Overview: Data extraction capabilities
2. Extraction Goals:
   - Goal format and syntax
   - Examples
   - Best practices
3. Extraction Schemas:
   - JSON Schema format
   - Properties definition
   - Required fields
   - Nested objects
   - Arrays
4. Examples by Type:
   - E-commerce: Product data
   - Forms: Form fields
   - Tables: Table data
   - Nested: Multi-level structures
5. Validation:
   - Schema validation
   - Type coercion
   - Required field handling
6. Complex Cases:
   - Repeated elements
   - Dynamic content
   - Conditional extraction
7. Accessing Results:
   - Response format
   - JSON structure
   - Nested data access
8. Debugging:
   - Common extraction issues
   - Improving accuracy
   - Cost vs accuracy

**Cross-References**:
- api-reference.md (TaskRequest.extraction_goal parameter)
- workflows.md (extraction in workflows)
- examples.md (extraction examples)

---

### 10. error-handling.md
**Type**: Guide  
**Priority**: MEDIUM  
**Length Target**: 350-400 lines

**context-index.md Sections**:
- Patterns & Conventions (L780-800) — Error handling approach
- Configuration (L630-680) — Timeout settings
- Key Concepts (L340-360) — Status codes

**Key Source Files to Reference**:
- `exceptions.py` — Custom exceptions
- `forge/agent.py` — Error handling in agent
- `forge/api_app.py` — Exception middleware
- `skyvern/config.py` — Timeout configs

**What to Include**:
1. Overview: Error types and categories
2. Common Errors:
   - TimeoutError: Page not loading
   - FailedStepError: Step execution failed
   - ElementNotFoundError: Locator failed
   - AuthenticationError: Credential issue
   - ValidationError: Bad input
3. HTTP Status Codes:
   - 400: Bad request
   - 401: Unauthorized
   - 403: Forbidden
   - 404: Not found
   - 500: Server error
4. Debugging Techniques:
   - Enable debug logging
   - VNC (virtual display)
   - Screenshot inspection
   - Step-by-step execution
5. Retry Strategies:
   - Automatic retries
   - Exponential backoff
   - Max retries configuration
6. Timeouts:
   - Page loading timeout
   - Element loading timeout
   - Operation timeout
   - Configuring timeouts
7. Recovery & Fallbacks:
   - Handling step failures
   - Alternative locators
   - Graceful degradation
8. Logging:
   - Log levels
   - Structured logging
   - Log analysis
9. Troubleshooting Guide:
   - "Task never completes"
   - "Element not found"
   - "LLM call fails"
   - "Extraction empty"

**Cross-References**:
- getting-started.md (first errors encountered)
- browser-automation.md (waiting strategies)
- deployment.md (logging setup)

---

### 11. deployment.md
**Type**: Guide  
**Priority**: MEDIUM  
**Length Target**: 400-450 lines

**context-index.md Sections**:
- Overview (L1-30) — Tech stack
- Build & Run (L702-750) — Deployment info
- Configuration (L630-680) — All env vars
- File Tree (L34-150) — Docker files

**Key Source Files to Reference**:
- `docker-compose.yml` — Local Docker setup
- `Dockerfile` — Backend image
- `Dockerfile.ui` — Frontend image
- `alembic/` — Database migrations
- `.env.example` — Environment template
- `.github/workflows/` — CI/CD pipelines

**What to Include**:
1. Overview: Deployment options
2. Local Development:
   - Docker Compose setup
   - Running services
   - Debugging
3. Environment Variables:
   - Complete reference
   - Which are required
   - Default values
4. Database Setup:
   - PostgreSQL installation
   - Connection string format
   - Alembic migrations
   - Running migrations
5. LLM Configuration:
   - OpenAI setup
   - Anthropic setup
   - Vertex AI setup
   - Setting API keys
6. Cloud Deployment:
   - If applicable
   - Configuration differences
   - Scaling considerations
7. Monitoring & Observability:
   - Logging
   - Metrics
   - Health checks
   - OpenTelemetry setup
8. Security:
   - SSL/TLS
   - Secret management
   - Network isolation
9. Troubleshooting:
   - Database connection issues
   - Port conflicts
   - LLM auth issues
   - Startup failures

**Cross-References**:
- getting-started.md (quick start alternative)
- configuration section in context-index.md (env vars)
- agents-and-ai.md (LLM setup)

---

### 12. cli-reference.md
**Type**: Reference  
**Priority**: LOW  
**Length Target**: 200-250 lines

**context-index.md Sections**:
- File Tree (L60-90) — cli/ directory
- Build & Run (L702-720) — CLI usage

**Key Source Files to Reference**:
- `cli/__init__.py` — CLI entry point
- `cli/quickstart.py` — Quickstart command
- `cli/run_commands.py` — Task/workflow execution
- `cli/setup_commands.py` — Setup commands
- `cli/commands/` — Individual commands
- `cli/core/` — CLI utilities

**What to Include**:
1. Overview: CLI purpose
2. Installation: How to get the CLI
3. Configuration: .env, config files
4. Commands:
   - `skyvern quickstart` — Initial setup
   - `skyvern run-task` — Run single task
   - `skyvern run-workflow` — Run workflow
   - `skyvern setup` — Setup steps
   - Other commands
5. For Each Command:
   - Syntax and arguments
   - Options and flags
   - Examples
   - Exit codes
6. Configuration Files:
   - Where they live
   - Format
   - Supported options
7. Environment Variables for CLI
8. Troubleshooting:
   - Command not found
   - Permission errors
   - Configuration issues

**Cross-References**:
- getting-started.md (quickstart command)
- deployment.md (env vars)

---

### 13. examples.md
**Type**: Reference  
**Priority**: LOW  
**Length Target**: 250-350 lines

**context-index.md Sections**:
- Key Concepts (L252-420) — Domain models for examples
- Architecture (L152-250) — How things work

**Key Source Files to Reference**:
- Look for example directories in context/skyvern/
- SDK examples in test files
- API examples in route documentation

**What to Include**:
1. Overview: Example categories
2. Basic Examples:
   - Visit URL and extract data
   - Fill and submit form
   - Click button
3. Authentication:
   - Login and then interact
   - Using stored credentials
   - Multi-step auth
4. Data Extraction:
   - E-commerce product data
   - Table scraping
   - Form field extraction
5. Complex Workflows:
   - Multi-page workflow
   - Conditional logic
   - Looping
6. API Examples:
   - Create and run task
   - Get task status
   - Create workflow
7. SDK Examples:
   - Python SDK usage
   - Using decorators
   - Workflow DSL
8. Error Handling:
   - Retrying failed tasks
   - Debugging extraction
9. Advanced:
   - Custom prompts
   - Using credentials
   - Browser session management

**Cross-References**:
- All other pages (e.g., "See workflows.md for details")
- source code via context-index.md

---

## Quick Summary Table

| Page | Type | Priority | Lines | Key Sections | Key Files |
|------|------|----------|-------|--------------|-----------|
| getting-started | Guide | - | 200 | Overview, Build&Run | quickstart.py, docker-compose.yml |
| architecture | Explanation | - | 200 | Overview, Architecture | agent.py, api_app.py |
| api-reference | Reference | HIGH | 300-400 | API Surface, Data Model | routes/*, schemas/* |
| sdk-guide | Guide | HIGH | 400-500 | Key Concepts, Architecture | __init__.py, workflow/*, decorators |
| workflows | Guide | HIGH | 400-500 | Key Concepts, Architecture | workflow/models/, workflow/service.py |
| agents-and-ai | Explanation | MEDIUM | 350-400 | Overview, Architecture, Key Concepts | agent.py, prompts.py, llm/* |
| browser-automation | Guide | MEDIUM | 350-400 | Key Concepts, Architecture | browser.py, page.py, actions.py |
| credentials-and-auth | Guide | MEDIUM | 350-400 | Architecture, Data Model | models.py, security.py, routes/credentials.py |
| data-extraction | Guide | MEDIUM | 350-400 | Key Concepts, Data Model | schemas/tasks.py, agent.py |
| error-handling | Guide | MEDIUM | 350-400 | Patterns & Conventions, Configuration | exceptions.py, agent.py, api_app.py |
| deployment | Guide | MEDIUM | 400-450 | Build & Run, Configuration | docker-compose.yml, alembic/, .env.example |
| cli-reference | Reference | LOW | 200-250 | File Tree, Build & Run | cli/* |
| examples | Reference | LOW | 250-350 | Key Concepts, Architecture | test examples, API examples |

---

**Last Updated**: 2026-03-18  
**Total Pages**: 13 (2 complete, 11 remaining)  
**Context File**: context/context-index.md (836 lines)

