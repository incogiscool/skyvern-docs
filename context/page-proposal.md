# Documentation Page Proposal

**Status**: Approved by developer

## Proposed Pages

| # | Path | Type | Description | Outcome |
|---|------|------|-------------|---------|
| 1 | getting-started | Guide | Installation (pip, Docker, source), quickstart (running first task), environment setup (LLM provider config, database) | Reader will be able to install Skyvern and run their first browser-automation task in 10 minutes. |
| 2 | architecture | Explanation | System overview and components, task execution flow (agent → LLM → browser → database), workflow engine and block types, database schema | Reader will understand how Skyvern's major components interact and how a task flows from API request through LLM and browser to the database. |
| 3 | api-reference | Reference | Complete REST API endpoints (tasks, workflows, browser sessions, credentials), request/response schemas with examples, status codes and error handling, WebSocket streaming API | Reader will be able to construct valid API requests and interpret responses for every available endpoint. |
| 4 | sdk-guide | Guide | Python SDK (Skyvern class, SkyvernBrowser, SkyvernBrowserPage), decorators and workflow DSL (@workflow, @action, @cached, @login), script service functions, TypeScript SDK basics | Reader will be able to use the Python and TypeScript SDKs to create tasks and workflows programmatically. |
| 5 | workflows | Guide | Workflow definition format, block types (ActionBlock, ValidationBlock, LoopBlock, etc.), parameters and variable passing, conditional logic and loops | Reader will be able to define multi-step workflows using Skyvern's block types and parameter system. |
| 6 | browser-automation | Guide | Using AI-powered locators, screenshot analysis and vision capabilities, browser session management, handling dynamic content and loading states | Reader will be able to configure and control AI-powered browser sessions for dynamic web content. |
| 7 | agents-and-ai | Explanation | How the agent system works, LLM provider support (OpenAI, Claude, Vertex AI), prompt engineering and customization, token usage and cost optimization | Reader will understand how Skyvern's agent reasons about pages and how to configure LLM providers. |
| 8 | credentials-and-auth | Guide | Managing credentials (API keys, passwords, OAuth), multi-tenancy and organization isolation, JWT authentication, webhook security | Reader will be able to securely store and use credentials within tasks and workflows. |
| 9 | data-extraction | Guide | Data extraction goals and schemas, JSON schema definition, handling complex nested data, examples (e-commerce, forms, tables) | Reader will be able to define JSON schemas and extract structured data from web pages. |
| 10 | error-handling | Guide | Common errors and troubleshooting, retry strategies and max steps, timeout configuration, debug mode and logging | Reader will be able to diagnose common failures and configure retry and timeout strategies. |
| 11 | deployment | Guide | Docker Compose setup, cloud deployment (if applicable), environment variables and configuration, database migrations, monitoring and observability | Reader will be able to deploy Skyvern with Docker Compose and configure it for a production environment. |
| 12 | cli-reference | Reference | Command reference (run-task, run-workflow, setup, etc.), options and flags, configuration files | Reader will be able to look up any CLI command, flag, and configuration option. |
| 13 | examples | Reference | Common workflows (form filling, data scraping, login), complex scenarios (multi-page workflows, error handling), code snippets for different use cases | Reader will be able to find and adapt working code examples for the most common automation scenarios. |


## Page Outcomes

**Page 1**: Reader will be able to install Skyvern and run their first browser-automation task in 10 minutes.

**Page 2**: Reader will understand how Skyvern's major components interact and how a task flows from API request through LLM and browser to the database.

**Page 3**: Reader will be able to construct valid API requests and interpret responses for every available endpoint.

**Page 4**: Reader will be able to use the Python and TypeScript SDKs to create tasks and workflows programmatically.

**Page 5**: Reader will be able to define multi-step workflows using Skyvern's block types and parameter system.

**Page 6**: Reader will be able to configure and control AI-powered browser sessions for dynamic web content.

**Page 7**: Reader will understand how Skyvern's agent reasons about pages and how to configure LLM providers and prompts.

**Page 8**: Reader will be able to securely store and use credentials within tasks and workflows.

**Page 9**: Reader will be able to define JSON schemas and extract structured data from web pages.

**Page 10**: Reader will be able to diagnose common failures and configure retry and timeout strategies.

**Page 11**: Reader will be able to deploy Skyvern with Docker Compose and configure it for a production environment.

**Page 12**: Reader will be able to look up any CLI command, flag, and configuration option.

**Page 13**: Reader will be able to find and adapt working code examples for the most common automation scenarios.

## Notes

- **Guides** (8 total): Practical, task-oriented documentation for users learning features
- **Explanations** (2 total): Conceptual documentation for understanding system design
- **References** (3 total): Lookup documentation (API, CLI, code examples)

## Approval

✅ **Approved** — Ready for documentation writer teams to create content based on the context index.
