# Documentation Page Proposal

**Status**: Approved by developer

## Proposed Pages

| # | Path | Type | Description | Outcome |
|---|------|------|-------------|---------|
| 1 | getting-started | Guide | Installation (pip, Docker, source), quickstart (running first task), environment setup (LLM provider config, database) | Reader will be able to install and run their first Skyvern task in 10 minutes |
| 2 | architecture | Explanation | System overview and components, task execution flow (agent → LLM → browser → database), workflow engine and block types, database schema | Reader will understand how Skyvern's components interact and how tasks flow through the system |
| 3 | api-reference | Reference | Complete REST API endpoints (tasks, workflows, browser sessions, credentials), request/response schemas with examples, status codes and error handling, WebSocket streaming API | Reader will be able to construct valid API requests and understand all available endpoints and their parameters |
| 4 | sdk-guide | Guide | Python SDK (Skyvern class, SkyvernBrowser, SkyvernBrowserPage), decorators and workflow DSL (@workflow, @action, @cached, @login), script service functions, TypeScript SDK basics |
| 5 | workflows | Guide | Workflow definition format, block types (ActionBlock, ValidationBlock, LoopBlock, etc.), parameters and variable passing, conditional logic and loops |
| 6 | browser-automation | Guide | Using AI-powered locators, screenshot analysis and vision capabilities, browser session management, handling dynamic content and loading states |
| 7 | agents-and-ai | Explanation | How the agent system works, LLM provider support (OpenAI, Claude, Vertex AI), prompt engineering and customization, token usage and cost optimization |
| 8 | credentials-and-auth | Guide | Managing credentials (API keys, passwords, OAuth), multi-tenancy and organization isolation, JWT authentication, webhook security |
| 9 | data-extraction | Guide | Data extraction goals and schemas, JSON schema definition, handling complex nested data, examples (e-commerce, forms, tables) |
| 10 | error-handling | Guide | Common errors and troubleshooting, retry strategies and max steps, timeout configuration, debug mode and logging |
| 11 | deployment | Guide | Docker Compose setup, cloud deployment (if applicable), environment variables and configuration, database migrations, monitoring and observability |
| 12 | cli-reference | Reference | Command reference (run-task, run-workflow, setup, etc.), options and flags, configuration files |
| 13 | examples | Reference | Common workflows (form filling, data scraping, login), complex scenarios (multi-page workflows, error handling), code snippets for different use cases |

## Notes

- **Guides** (8 total): Practical, task-oriented documentation for users learning features
- **Explanations** (2 total): Conceptual documentation for understanding system design
- **References** (3 total): Lookup documentation (API, CLI, code examples)

## Approval

✅ **Approved** — Ready for documentation writer teams to create content based on the context index.
