# Open Questions

These are areas of uncertainty discovered during codebase exploration that may need deeper investigation or clarification.

## Architecture & Design

1. **Streaming API Architecture**: The `forge/sdk/routes/streaming/` directory contains multiple channel types (CDP, execution, exfiltration, VNC, screenshot) in `streaming/channels/`. What is the purpose of each channel type? When is each used? How do they interact?

2. **Workflow Block Recursion**: How does `WorkflowService` handle nested/recursive blocks (e.g., a LoopBlock containing ActionBlocks containing ValidationBlocks)? Is there a visitor pattern, recursive traversal, or linearized execution queue?

3. **Task V1 vs V2 Architecture**: The codebase has both `task_v1_service.py` and `task_v2_service.py`. What are the architectural differences between them? When is each used? Is V2 the recommended path forward?

4. **Browser Session Persistence**: How are persistent browser sessions maintained across task executions? Is there a `BrowserSessionManager` singleton? What's the cleanup strategy for stale sessions?

5. **Observability Implementation**: The `forge/api_app.py` imports `OTELSetup` from `cloud.observability.otel_setup` (which is not in this repo). How is OpenTelemetry integrated? What metrics and traces are exported in production?

## Authentication & Multi-tenancy

6. **Multi-tenancy Enforcement**: Database models include `organization_id` fields. How is multi-tenancy enforced at the API level? Is there middleware that validates the requesting user's organization context?

7. **Credential Encryption**: How are stored credentials (passwords, API keys, OAuth tokens) encrypted at rest? What's the encryption key derivation strategy? Is there a key rotation mechanism?

8. **JWT Scope & Claims**: What claims are included in JWT tokens beyond expiration? Are there role-based scopes (admin, user, viewer)?

## LLM & AI

9. **LLM Provider Selection**: How does `LLMConfigRegistry` work? Can users dynamically switch between OpenAI, Claude (Anthropic), Vertex AI, and other providers? Is there a provider-specific configuration layer?

10. **Vision LLM Calls**: The agent calls vision LLMs for screenshot analysis. How are screenshots preprocessed (resizing, compression) before sending to the LLM? Are there any cost optimization strategies?

11. **Prompt Engineering**: The `prompts.py` file uses a prompt engine. Is there a template system? How are prompts versioned? Can users customize them?

12. **Token Counting**: Token usage is tracked in `StepModel` (input_token_count, output_token_count, reasoning_token_count). How are these calculated for different LLM providers with different tokenization schemes?

## Database & Persistence

13. **Database Migrations**: The `alembic/versions/` directory has many migration files. Is there a strategy for running them in CI/CD? Are there data migration utilities for evolving the schema?

14. **Read Replica Support**: The `DATABASE_REPLICA_STRING` config suggests read replica support. How are reads routed to replicas? Is there automatic failover?

15. **Index Strategy**: Various tables have indexes on organization_id, created_at, etc. Are there documented index guidelines for new tables? How is query performance monitored?

## Frontend & State Management

16. **React State Management**: What state management solution does the React frontend use? Redux, Zustand, Context API, or something else? How is real-time data synchronized with the backend?

17. **WebSocket vs SSE**: The API supports both WebSocket (streaming) and SSE (events). Why both? When is each used from the frontend perspective?

18. **Frontend Error Handling**: How are API errors displayed to users? Is there a centralized error handler or per-component error boundaries?

## CLI & Local Development

19. **Quickstart Command**: The `quickstart.py` handles initial setup. Does it guide users through LLM provider configuration? Does it launch Docker services automatically?

20. **MCP Tools**: The `cli/mcp_tools/` implements Model Context Protocol tools. What's the intended use case? Is this for AI-assisted development?

## Deployment & Operations

21. **CI/CD Pipeline**: The `.github/workflows/` directory has files like `auto-release.yml`, `publish-fern-docs.yml`, `sdk-release.yml`. What's the full deployment pipeline? How are cloud vs. local deployments differentiated?

22. **Feature Flags**: The codebase has `experimentation/` and settings like `ENABLE_EXP_ALL_TEXTUAL_ELEMENTS_INTERACTABLE`. What's the feature flag system? How are experiments tracked?

23. **Logging & Monitoring**: Beyond OpenTelemetry, what monitoring is in place? Are there dashboards for task success rates, LLM usage, performance metrics?

## Edge Cases & Error Handling

24. **Timeout Handling**: There are many timeout configs (BROWSER_LOADING_TIMEOUT_MS, OPTION_LOADING_TIMEOUT_MS, etc.). How are timeouts handled? Do they gracefully degrade or hard fail?

25. **Webhook Retry Logic**: When a webhook callback fails, is there a retry mechanism? Exponential backoff? Dead-letter queue?

26. **File Download Handling**: The agent handles file downloads. How are large files handled? Is there a size limit? Are downloads streamed or buffered?

## Type System & Validation

27. **Custom Validators**: Pydantic models use custom validators (e.g., `TaskRequest.validate_url()`). Are these validators documented? What validation rules exist?

28. **Enum Extensibility**: How can the set of `TaskStatus`, `StepStatus`, or `ProxyLocationInput` values be extended without code changes?

## Testing & Quality

29. **Test Coverage**: What's the test coverage percentage? Are there coverage gates in CI?

30. **Integration Test Strategy**: Do integration tests use real Playwright browsers or mocked? Do they use real LLM calls or mocked?

## Documentation & Examples

31. **Documentation Source**: This repository seems to be a documentation generation project (based on git history). What's the source of truth for API docs? Is there an OpenAPI/Swagger spec?

32. **Code Examples**: Are there examples for common workflows (form submission, data extraction, etc.) in a separate examples directory?

---

## Investigation Priority

**High Priority** (needed for initial documentation):
- Questions 1-3 (architecture specifics)
- Question 6 (multi-tenancy)
- Question 21 (deployment pipeline)

**Medium Priority** (needed for comprehensive guides):
- Questions 4-5, 8-11, 19 (features and capabilities)
- Questions 13-15, 23 (operations)

**Low Priority** (nice-to-have):
- Questions 7, 16-18, 24-32 (implementation details)

