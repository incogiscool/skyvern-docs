# Troubleshooting

## Browser fails to start

**What you see:** The backend starts but tasks immediately fail with `FailedToTakeScreenshot` or `MissingBrowserState`. Logs show errors like `Xvfb failed to start`.

**Why it happens:** The headful browser requires a display. In Docker or Linux environments without a graphical display, Xvfb (virtual framebuffer) must be running.

**Fix:** The official Docker image starts Xvfb automatically via the entrypoint script. If you're running outside Docker, start Xvfb manually:

```bash
Xvfb :99 -screen 0 1920x1080x16 &
export DISPLAY=:99
```

Or set `BROWSER_TYPE=chromium-headless` if your workflow doesn't require a real display.

---

## Tasks time out before completing

**What you see:** Tasks finish with status `timed_out`. The run timeline shows step count hitting the limit.

**Why it happens:** The task ran out of steps before completing the goal. The default limit is `MAX_STEPS_PER_RUN=10`.

**Fix:** Increase the step limit either globally or per-task:

```bash
# Global: set in .env
MAX_STEPS_PER_RUN=25
```

```json
// Per-task: pass in the request body
{"prompt": "...", "max_steps": 25}
```

---

## LLM returns errors or rate limit responses

**What you see:** Steps fail with `LLMProviderError`. Logs show 429 or 503 responses from the LLM API.

**Why it happens:** You've hit the rate limit for your LLM provider, or the API key is invalid.

**Fix:** Check your API key is set correctly:

```bash
# In .env - must match the ENABLE_ flag
ENABLE_OPENAI=true
LLM_KEY=OPENAI_GPT4O
OPENAI_API_KEY=sk-...
```

For rate limits, either upgrade your API tier or add a secondary LLM with `SECONDARY_LLM_KEY` as a fallback.

---

## Database connection errors

**What you see:** The backend fails to start or requests return 500 errors. Logs show `asyncpg.exceptions.TooManyConnectionsError` or connection refused errors.

**Why it happens:** PostgreSQL is not running, the connection string is wrong, or the connection pool is exhausted.

**Fix for connection refused:** Verify PostgreSQL is running and the `DATABASE_STRING` is correct:

```bash
# Default local connection string
DATABASE_STRING=postgresql+psycopg://skyvern@localhost/skyvern
```

**Fix for too many connections:** Reduce pool size or increase PostgreSQL's `max_connections`:

```bash
DATABASE_POOL_SIZE=3
DATABASE_POOL_MAX_OVERFLOW=5
```

---

## Migrations fail on startup

**What you see:** The Docker container exits immediately. Logs show `alembic.util.exc.CommandError` or a migration conflict.

**Why it happens:** The database schema is out of date or in a conflicting state.

**Fix:** Run migrations manually to see the full error:

```bash
python -m alembic upgrade head
python -m alembic current
```

If the database is at an unknown revision, check `alembic_version` in PostgreSQL:

```sql
SELECT * FROM alembic_version;
```

---

## Credentials not found during task execution

**What you see:** A task that uses stored credentials fails with `ContextParameterValueNotFound` or the agent reports it couldn't find login credentials.

**Why it happens:** The `parameter_keys` list in the block doesn't reference the credential parameter key, or the credential ID is wrong.

**Fix:** Make sure the workflow parameter `key` matches what's listed in `parameter_keys`:

```yaml
parameters:
  - parameter_type: credential
    key: my_creds       # <-- this key
    credential_id: cred_abc123

blocks:
  - block_type: login
    label: sign_in
    parameter_keys: [my_creds]   # <-- must match
```

---

## Artifact signed URLs return 403

**What you see:** Artifact URLs returned by the API return `Access Denied` or 403 when accessed.

**Why it happens:** Signed URLs expire after 24 hours (controlled by `PRESIGNED_URL_EXPIRATION`). The URL was generated before the current request.

**Fix:** Re-fetch the artifact from the API to get a fresh signed URL:

```bash
GET /api/v1/artifacts/{artifact_id}
```

---

## API key is rejected (401)

**What you see:** All API requests return `{"detail": "Unauthorized"}`.

**Why it happens:** The API key in the `x-api-key` header doesn't match any organization in the database, or the local key has been regenerated.

**Fix for local deployments:** Find the current key in the UI (Settings page) or regenerate it:

```bash
# From the Skyvern UI or via the internal auth endpoint
POST /api/v1/internal/auth/repair
```

**Fix for Docker:** Check `.streamlit/secrets.toml` - the API key is written there on first startup.

---

## CORS errors in browser clients

**What you see:** Browser requests fail with `Access to fetch at ... has been blocked by CORS policy`.

**Why it happens:** The `ALLOWED_ORIGINS` setting doesn't include your frontend's origin.

**Fix:** Add your origin to the allowed list:

```bash
ALLOWED_ORIGINS=["https://my-app.example.com","http://localhost:3000"]
```

The default `["*"]` allows all origins, which is fine for development but should be restricted in production.
