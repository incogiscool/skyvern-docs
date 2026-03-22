# Developer Answers

## Q: **`skyvern-ts/client/`** — This directory exists but only has a `client/` subdirectory with a TypeScript SDK. It appears to be a separate Fern-generated TypeScript SDK. Its package.json and full structure weren't explored. How is it published/versioned relative to the Python SDK?

Completely forget about the docs and fern folders!!!!

## Q: **Script system** — There are `ScriptModel`, `ScriptFileModel`, `WorkflowScriptModel`, `ScriptBlockModel` in the DB. The scripts system appears to be for AI-generated Playwright scripts for caching/speeding up runs. The full flow (generation, caching, fallback) wasn't traced in detail. Where is `generate_script.py` (in `skyvern/core/script_generations/`) invoked in the main execution loop?

*(skipped)*

## Q: **`TaskRunModel` vs `TaskModel`** — There appear to be two task-like tables: `tasks` (TaskModel, legacy v1) and `task_v2` (TaskV2Model). The `task_runs` table (TaskRunModel at L957) seems to be a unified run tracking table. What exactly is the relationship?

*(skipped)*

## Q: **Debug sessions** — `DebugSessionModel` exists (with VNC support). How are these exposed to users? Is there a UI for debug sessions?

*(skipped)*

## Q: **Observer/Cruise** — There are alembic migrations referencing `ObserverCruise` and `observer_thoughts` (ThoughtModel at L864). These appear to be the v2 task execution chain. The `cruise` concept maps to what in the current API? Is it fully superseded by TaskV2/skyvern-2.0?

*(skipped)*

## Q: **`publish_workflow` parameter** — `run_task()` accepts a `publish_workflow` flag (only for skyvern-2.0). What does this do exactly? Does it auto-save the task as a reusable workflow?

*(skipped)*

## Q: **`run_with` field** — Both `TaskRunRequest` and `WorkflowRunRequest` have a `run_with` field. What does this control?

*(skipped)*

## Q: **Sequential workflow setting** — There's a `2025_09_18_0529-8f208daee7d1_add_workflow_sequential_setting.py` migration. What does this setting do?

*(skipped)*

## Q: **AI Fallback** — `ai_fallback` field on workflows and workflow runs (migrations reference it). When is AI fallback triggered? Is this the script-to-AI fallback mechanism?

*(skipped)*

## Q: **Bitwarden CLI server** — The `/source/bitwarden-cli-server/` directory exists. How is it deployed (Docker image? subprocess?)? What's the communication protocol?

*(skipped)*

## Q: **VNC streaming** — Port 6080 is exposed in docker-compose for "VNC WebSocket streaming." The streaming routes exist in `skyvern/forge/sdk/routes/streaming/`. Is this live browser streaming to the UI?

*(skipped)*

## Q: **`BlockRunModel`** (L1038) vs `WorkflowRunBlockModel` (L757) — Two block-run related models. What's the distinction?

*(skipped)*

## Q: **Consistent Workflows** — `fern/workflows/consistent-workflows.mdx` exists in nav but its content wasn't read. What caching/consistency mechanisms does it document?

*(skipped)*

## Q: **`run_with` field in workflows** — The DOC-PLAN mentions this feature under API/SDK but it's not yet documented.

*(skipped)*

## Q: **`include_action_history_in_verification`** — Task parameter that was added but its effect on behavior wasn't explored in detail.

*(skipped)*

## Q: **`max_screenshot_scrolls`** — Parameter for controlling viewport screenshot scrolling. Its interaction with task output wasn't fully explored.

*(skipped)*

## Q: **AI Suggestions** — `AISuggestionModel` (L692) exists. What does it do? Is it used in the workflow copilot?

*(skipped)*

## Q: **Workflow Copilot** — `WorkflowCopilotChatModel` and `WorkflowCopilotChatMessageModel` exist. Is this the AI-assisted workflow building feature? Is it documented?

*(skipped)*

## Q: **`login()` method** — The SDK client has a `login()` method (L1727). This seems different from credential-based login. What does it do (OAuth flow? API key generation?)?

*(skipped)*

## Q: **TypeScript SDK** — `@skyvern/client` is mentioned in `fern/docs.yml` snippets config. Where is it located (`skyvern-ts/client/`?)? What's its public API surface?

Again, completely forget about fern and docs folders! They are outdated.
