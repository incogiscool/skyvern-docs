# Open Questions

## Architecture

1. **`skyvern-ts/client/`** — This directory exists but only has a `client/` subdirectory with a TypeScript SDK. It appears to be a separate Fern-generated TypeScript SDK. Its package.json and full structure weren't explored. How is it published/versioned relative to the Python SDK?

2. **Script system** — There are `ScriptModel`, `ScriptFileModel`, `WorkflowScriptModel`, `ScriptBlockModel` in the DB. The scripts system appears to be for AI-generated Playwright scripts for caching/speeding up runs. The full flow (generation, caching, fallback) wasn't traced in detail. Where is `generate_script.py` (in `skyvern/core/script_generations/`) invoked in the main execution loop?

3. **`TaskRunModel` vs `TaskModel`** — There appear to be two task-like tables: `tasks` (TaskModel, legacy v1) and `task_v2` (TaskV2Model). The `task_runs` table (TaskRunModel at L957) seems to be a unified run tracking table. What exactly is the relationship?

4. **Debug sessions** — `DebugSessionModel` exists (with VNC support). How are these exposed to users? Is there a UI for debug sessions?

5. **Observer/Cruise** — There are alembic migrations referencing `ObserverCruise` and `observer_thoughts` (ThoughtModel at L864). These appear to be the v2 task execution chain. The `cruise` concept maps to what in the current API? Is it fully superseded by TaskV2/skyvern-2.0?

## Features

6. **`publish_workflow` parameter** — `run_task()` accepts a `publish_workflow` flag (only for skyvern-2.0). What does this do exactly? Does it auto-save the task as a reusable workflow?

7. **`run_with` field** — Both `TaskRunRequest` and `WorkflowRunRequest` have a `run_with` field. What does this control?

8. **Sequential workflow setting** — There's a `2025_09_18_0529-8f208daee7d1_add_workflow_sequential_setting.py` migration. What does this setting do?

9. **AI Fallback** — `ai_fallback` field on workflows and workflow runs (migrations reference it). When is AI fallback triggered? Is this the script-to-AI fallback mechanism?

## Infrastructure

10. **Bitwarden CLI server** — The `/source/bitwarden-cli-server/` directory exists. How is it deployed (Docker image? subprocess?)? What's the communication protocol?

11. **VNC streaming** — Port 6080 is exposed in docker-compose for "VNC WebSocket streaming." The streaming routes exist in `skyvern/forge/sdk/routes/streaming/`. Is this live browser streaming to the UI?

12. **`BlockRunModel`** (L1038) vs `WorkflowRunBlockModel` (L757) — Two block-run related models. What's the distinction?

## Documentation Gaps

13. **Consistent Workflows** — `fern/workflows/consistent-workflows.mdx` exists in nav but its content wasn't read. What caching/consistency mechanisms does it document?

14. **`run_with` field in workflows** — The DOC-PLAN mentions this feature under API/SDK but it's not yet documented.

15. **`include_action_history_in_verification`** — Task parameter that was added but its effect on behavior wasn't explored in detail.

16. **`max_screenshot_scrolls`** — Parameter for controlling viewport screenshot scrolling. Its interaction with task output wasn't fully explored.

17. **AI Suggestions** — `AISuggestionModel` (L692) exists. What does it do? Is it used in the workflow copilot?

18. **Workflow Copilot** — `WorkflowCopilotChatModel` and `WorkflowCopilotChatMessageModel` exist. Is this the AI-assisted workflow building feature? Is it documented?

## SDK / Client

19. **`login()` method** — The SDK client has a `login()` method (L1727). This seems different from credential-based login. What does it do (OAuth flow? API key generation?)?

20. **TypeScript SDK** — `@skyvern/client` is mentioned in `fern/docs.yml` snippets config. Where is it located (`skyvern-ts/client/`?)? What's its public API surface?
