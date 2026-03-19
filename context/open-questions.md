# Open Questions

These are areas that were unclear during exploration or require deeper investigation:

## Architecture & Design

1. **RBAC (Role-Based Access Control)**
   - Currently all organization members have full access to all resources
   - Are there plans for role-based access (admin, editor, viewer roles)?
   - Is there any user-level isolation per org?
   - **Impact:** Affects documentation of permissions and security best practices

2. **Database Replica String**
   - `DATABASE_REPLICA_STRING` exists in config.py but no code found using it
   - Are read/write operations actually split across replica/primary?
   - Is replica replication/failover documented?
   - **Impact:** Affects documentation of deployment architecture and HA setup

3. **Workflow Triggering Mechanisms**
   - `/workflows/{workflow_id}/run` is found (manual execution)
   - No visible scheduled/cron-triggered workflow endpoints
   - No visible event-triggered workflow setup
   - Are webhooks incoming → workflow trigger?
   - **Impact:** Critical for users understanding automation capabilities

4. **State Persistence & Server Restarts**
   - For long-running workflows/tasks, how is state persisted if server restarts?
   - Are in-flight tasks resumed or aborted?
   - Is there a queue (Temporal, Celery, custom)?
   - **Impact:** Affects documentation of reliability guarantees

5. **Concurrency & Throttling**
   - What's the max concurrent tasks per organization?
   - Per-user limits?
   - How are browser resources managed at scale?
   - Is there a queue system?
   - **Impact:** Critical for SLA and performance documentation

## Data & Persistence

6. **Artifact Retention & Cleanup**
   - `cleanup_service.py` exists but triggers and retention policy unclear
   - How long are artifacts kept?
   - Is cleanup policy configurable per org?
   - Are there cost implications (S3 storage)?
   - **Impact:** Affects operational cost documentation

7. **Workflow Definition Versioning**
   - `WorkflowVersionModel` likely exists in database
   - How many versions retained?
   - Can users rollback to previous workflow versions?
   - Is version history exposed in API?
   - **Impact:** Affects workflow editing workflows documentation

8. **Script Caching Mechanism**
   - Script results are mentioned as cached
   - What's the cache key (inputs only? full script + inputs)?
   - Cache TTL?
   - Is this opt-in or default?
   - **Impact:** Affects documentation of script behavior & performance

## Authentication & Security

9. **WebSocket Authentication**
   - Streaming channels (VNC, CDP) use token-based auth
   - Is this the same JWT from API?
   - Are there separate token types/scopes?
   - Token expiration for long-running streams?
   - **Impact:** Affects security documentation for streaming endpoints

10. **Credential Encryption & Key Management**
    - Credentials encrypted with Fernet cipher
    - Where are encryption keys stored?
    - Key rotation mechanism?
    - Is key management integrated with org secret stores?
    - **Impact:** Affects security & compliance documentation

11. **CORS & Origin Verification**
    - `ALLOWED_ORIGINS` defaults to ["*"]
    - Is this correct default for security?
    - Should documentation recommend stricter settings?
    - **Impact:** Affects production deployment security guidance

## Integration & Extensions

12. **Browser Extension System**
    - `EXTENSIONS_BASE_PATH` and `EXTENSIONS` config exists
    - How are browser extensions loaded?
    - What's the supported extension interface?
    - Use cases beyond built-in functionality?
    - **Impact:** Affects extensibility documentation

13. **LLM Provider Coverage & Testing**
    - LiteLLM supports 100+ models
    - Which ones are actually tested/supported?
    - Fallback behavior if primary LLM fails?
    - **Impact:** Affects LLM selection documentation

14. **Integration Points (Make, Langchain, Llama Index)**
    - `integrations/` directory has Make, Langchain, LlamaIndex, MCP
    - Are these maintained as official integrations?
    - Version compatibility?
    - Documentation location?
    - **Impact:** Affects integration documentation strategy

15. **MCP (Model Context Protocol) Integration**
    - `cli/mcp.py` and `integrations/mcp/` exist
    - What are the capabilities?
    - Is this for LLM function calling or desktop integration?
    - User documentation?
    - **Impact:** Affects feature documentation completeness

## Organization & Project Structure

16. **Workspace/Team/Multi-tenancy Model**
    - `session.json` has `projectName` field
    - Is there a workspace concept above organization?
    - Can one user belong to multiple workspaces?
    - How is billing/metering organized?
    - **Impact:** Affects documentation of account structure

17. **API Versioning Strategy**
    - base_router (v1), legacy_base_router, legacy_v2_router
    - What's the v2 vs v1 difference?
    - Deprecation timeline for legacy routes?
    - Is there a v3 planned?
    - **Impact:** Affects API documentation structure

18. **Cloud vs Self-Hosted Architecture**
    - Session.json references `incogiscool/skyvern-docs` repo
    - Is Skyvern Cloud separate codebase?
    - What features are cloud-only?
    - Self-hosted deployment doc completeness?
    - **Impact:** Affects deployment documentation scope

## Performance & Observability

19. **Performance Monitoring**
    - PostHog integration exists
    - What metrics are tracked?
    - Are there performance SLAs?
    - How are slow tasks identified?
    - **Impact:** Affects monitoring/observability documentation

20. **Caching Strategy Beyond Scripts**
    - Redis is configured but specific usage unclear
    - What uses Redis (sessions, locks, cache)?
    - Cache invalidation strategy?
    - **Impact:** Affects scalability documentation

## Testing & Quality

21. **Test Coverage**
    - `tests/` directory exists with unit/SDK/smoke tests
    - What's the coverage percentage?
    - Are integration tests against real websites?
    - CI/CD pipeline details?
    - **Impact:** Affects quality/reliability documentation

22. **Evaluation Framework**
    - `evaluation/` directory has WebVoyager benchmark results
    - How frequently is evaluation run?
    - How are improvements measured?
    - Baseline comparisons?
    - **Impact:** Affects capability claims documentation

## Deployment & Operations

23. **Kubernetes Deployment**
    - `kubernetes-deployment/` directory exists
    - Is this production-ready?
    - Horizontal scaling strategy?
    - StatefulSet vs Deployment?
    - **Impact:** Affects K8s deployment documentation

24. **Observability & Logging**
    - Structlog is used for structured logging
    - JSON_LOGGING flag exists
    - What's the structured log schema?
    - Log aggregation recommendations?
    - **Impact:** Affects operations/monitoring documentation

25. **Database Migrations**
    - Alembic for migrations
    - Are migrations tested?
    - Rollback procedure documented?
    - Zero-downtime migration strategy?
    - **Impact:** Affects database upgrade documentation

