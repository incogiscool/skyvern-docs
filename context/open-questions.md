# Open Questions

## Architecture Questions

1. **`skyvern/services/` directory** — The CLAUDE.md mentions `skyvern/services/` for business logic, but this directory mainly contains `browser_recording/`. The actual business logic (tasks, workflows, runs) lives in `skyvern/forge/sdk/`. Is there a planned migration or is this intentional?

2. **`run_service` reference** — Multiple route handlers reference `run_service` but the exact file path wasn't confirmed (likely `skyvern/forge/sdk/services/run_service.py` or similar). Needs verification.

3. **Script Generation ("Code Run" feature)** — There are references to `generate_script`, `run_with` (`agent`, `code`, `code_v2`), `ai_fallback`, and `adaptive_caching` in workflow models. The full flow of the "code mode" vs "agent mode" wasn't deeply explored.

4. **WorkflowCopilot** — There's a `workflow_copilot.py` route and `WorkflowCopilotChat`/`WorkflowCopilotChatMessage` models in the DB. This appears to be an AI assistant for building workflows. How exactly it works wasn't explored.

5. **Observer/Cruise** — The DB has `ThoughtModel`, `observer_cruise_id`, `observer_thought_id` columns and migration files mention "ObserverCruise". This appears to be the TaskV2 internal planning mechanism. The relationship between `Task`, `TaskV2`, and `Observer*` wasn't fully traced.

6. **Evaluation framework** — `/source/evaluation/` directory exists but wasn't explored. This likely contains the WebVoyager benchmark evaluation code.

7. **Block execution engine** — Whether blocks execute sequentially or can run in parallel (the `run_sequentially` flag on WorkflowModel and `WorkflowService._execute_workflow_blocks_dag()` method suggest DAG-based execution).

## Documentation Questions

8. **TaskV2 vs Task (v1)** — From a user perspective, the difference between `skyvern-1.0` (task v1 flow) and `skyvern-2.0` (task v2 / Observer flow) should be clarified. The API surface exposes a unified `POST /v1/run/tasks` with an `engine` parameter, but internally they use very different code paths.

9. **CUA engines** — The OpenAI CUA and Anthropic CUA engines use a completely different execution path than Skyvern v1/v2. How they're integrated at the agent level wasn't traced.

10. **`publish_workflow` in TaskRunRequest** — The `publish_workflow: bool` field (only for skyvern-2.0) saves a successful task as a reusable workflow. The mechanics of this aren't documented.

11. **Debug Sessions** — The `DebugSessionModel` and debug session API were briefly explored but the full debug workflow (VNC streaming, block outputs, etc.) wasn't mapped.

12. **Script Blocks** — `ScriptBlockModel` references "script blocks" which appear to be cached/pre-generated code for workflow execution. The relationship between `ScriptModel`, `ScriptBlockModel`, `WorkflowScriptModel` needs clarification.

## Configuration Questions

13. **`ENABLE_VERTEX_AI`** — Referenced in `config.py` for Gemini via Vertex AI, but wasn't found in `.env.example`. Is this a cloud-only feature?

14. **`ENABLE_BEDROCK_ANTHROPIC`** — Similar to Vertex AI — referenced in model mappings but not in `.env.example`.

15. **`BLOCKED_HOSTS` and `ALLOWED_HOSTS`** — These settings exist in config but their enforcement mechanism wasn't traced.

## Integration Questions

16. **Temporal** — `WorkflowScheduleModel` has a `temporal_schedule_id` column suggesting Temporal.io integration for scheduled workflows. This wasn't explored and may be cloud-only.

17. **Bitwarden CLI server** — There's a `/source/bitwarden-cli-server/` directory that provides a local Bitwarden CLI wrapper. The integration details weren't fully explored.

18. **1Password SDK** — The `onepassword-sdk==0.3.0` dependency and `OnePasswordCredentialParameter` suggest 1Password integration. The setup (`OP_SERVICE_ACCOUNT_TOKEN`) is in `.env.example` but the implementation details weren't traced.

## Testing Questions

19. **Integration test setup** — `tests/sdk/` contains SDK tests but the setup requirements weren't examined. Do they need a running Skyvern instance?

20. **Browser test infrastructure** — How smoke tests (`tests/smoke_tests/`) are configured and what they cover wasn't explored.
