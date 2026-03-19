# Monitoring & Observability

This page covers how to configure logging, track performance metrics, and debug task executions in Skyvern.

## Logging

Skyvern uses [structlog](https://www.structlog.org/) for structured logging throughout the backend.

**Log level** is controlled by the `LOG_LEVEL` environment variable:

```
LOG_LEVEL=INFO   # default: INFO, DEBUG, WARNING, ERROR
```

**Structured JSON logs** (useful for log aggregators like Datadog or ELK):

```
JSON_LOGGING=true
```

When `JSON_LOGGING=false` (the default), logs are emitted as human-readable text with key-value pairs. When `true`, each line is a JSON object suitable for machine parsing.

**Log path** for file-based logs:

```
LOG_PATH=./log
```

The entrypoint script writes x11vnc errors to `/data/log/x11vnc.err`. Application logs go to stdout and are captured by your container runtime.

## Debug mode

Setting `DEBUG_MODE=true` enables verbose step-level logging and disables certain optimizations. Use it when diagnosing a specific task failure:

```
DEBUG_MODE=true
```

In debug mode, additional LLM request/response detail is logged at each step.

To log raw API request bodies (useful for auditing inbound payloads):

```
LOG_RAW_API_REQUESTS=true
```

## Analytics and performance monitoring

Skyvern integrates with [PostHog](https://posthog.com/) for anonymized usage analytics. This is enabled by default but collects only aggregated, non-personal data (task counts, engine usage, error rates).

To disable telemetry:

```
SKYVERN_TELEMETRY=false
```

To use your own PostHog project for monitoring:

```
POSTHOG_PROJECT_API_KEY=phc_yourkey
POSTHOG_PROJECT_HOST=https://app.posthog.com
```

## Debugging task runs

Every task produces a set of artifacts you can inspect after the fact:

- `screenshot_action` - screenshot taken after each browser action
- `screenshot_llm` - screenshot that was sent to the LLM at each step
- `llm_response` - raw LLM output for every step
- `html_scrape` - HTML snapshot of the page at each step
- `har` - complete HTTP Archive of all network requests
- `trace` - Playwright trace file (open with `playwright show-trace`)
- `recording` - full video recording of the browser session

Retrieve them for any run:

```bash
GET /api/v1/runs/{run_id}/artifacts
```

The response includes signed URLs valid for 24 hours.

### Streaming the live browser

Watch a running task in real time via the VNC WebSocket stream:

```
ws://localhost:8000/api/v1/stream/vnc/task/{task_id}?apikey=YOUR_KEY
```

The Skyvern UI uses this to render the live browser view. You can connect any VNC-compatible viewer to the same stream.

### Run timeline

For workflow runs and task_v2 runs, the timeline endpoint shows a structured breakdown of what happened at each stage:

```bash
GET /api/v1/runs/{run_id}/timeline
```

This is the fastest way to understand which block failed and why.

## Key metrics to watch

- **Task completion rate**: ratio of `completed` to `failed` + `terminated` statuses
- **Step count per task**: high step counts indicate the agent is struggling; check `MAX_STEPS_PER_RUN`
- **LLM latency**: visible in step timing within the run timeline
- **Database connection pool**: if queries are timing out, increase `DATABASE_POOL_SIZE` or `DATABASE_POOL_MAX_OVERFLOW`
- **Artifact storage**: ensure S3 or local disk has sufficient capacity; artifacts are not automatically cleaned up by default
