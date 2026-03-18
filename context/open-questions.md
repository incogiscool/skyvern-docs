# Open Questions

These are the key unresolved areas discovered during codebase exploration that may require developer clarification or deeper investigation before documentation can be completed.

1. **Task V1 vs V2**: The codebase has both `task_v1_service.py` and `task_v2_service.py`. Which is the recommended path for new users, and what are the key differences? This affects how the getting-started and api-reference pages should present task creation.

2. **Credential & Secret Encryption**: How are stored credentials (passwords, API keys, OAuth tokens) encrypted at rest, and is there a key rotation mechanism? Needed for the credentials-and-auth guide.

3. **Deployment & Observability**: The `forge/api_app.py` imports `OTELSetup` from `cloud.observability.otel_setup` (not in this repo). What's the full production deployment pipeline, and are there hosted/cloud-managed options beyond the Docker Compose quickstart?

4. **LLM Provider Configuration**: Can users dynamically switch between OpenAI, Anthropic/Claude, and Vertex AI at runtime, or is the provider fixed at startup via environment variables? Needed for the agents-and-ai and getting-started pages.

5. **Streaming Channels Purpose**: The `forge/sdk/routes/streaming/channels/` directory contains CDP, execution, exfiltration, VNC, and screenshot channels. Which channels are exposed as public API vs. internal-only, and what are their intended use cases?
