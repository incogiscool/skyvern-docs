# API Reference

All Skyvern API endpoints are served under `/api/v1`. Authentication is required for every endpoint — pass your API key in the `x-api-key` header.

```
x-api-key: YOUR_API_KEY
```

The base URL is `http://localhost:8000/api/v1` for a local deployment and `https://api.skyvern.com/api/v1` for Skyvern Cloud.

---

## Tasks

### Run a task

```
POST /run/tasks
```

Creates a task and starts execution immediately. The response is returned once the task is submitted to the queue; poll `GET /runs/{run_id}` to check completion.

**Request body:**

| Field | Type | Default | Description |
|---|---|---|---|
| `prompt` | string | required | Natural language description of what to do |
| `url` | string | `null` | Starting URL. If omitted, Skyvern infers it from the prompt |
| `engine` | string | `skyvern-2.0` | Execution engine: `skyvern-1.0`, `skyvern-2.0`, `openai-cua`, `anthropic-cua`, `ui-tars` |
| `data_extraction_schema` | object/array/string | `null` | JSON Schema for structured output |
| `proxy_location` | string/object | `RESIDENTIAL` | Proxy region (e.g. `US-CA`, `RESIDENTIAL_GB`) or a `GeoTarget` object |
| `max_steps` | integer | `null` | Maximum steps before the task times out |
| `webhook_url` | string | `null` | URL to POST the result to when the task finishes |
| `browser_session_id` | string | `null` | Reuse an existing persistent browser session |
| `totp_identifier` | string | `null` | Identifier for receiving TOTP codes (email or phone) |
| `totp_url` | string | `null` | URL Skyvern polls to get TOTP verification codes |
| `extra_http_headers` | object | `null` | Additional HTTP headers injected into all browser requests |
| `browser_address` | string | `null` | CDP URL to connect to an external browser |

**Example:**

```json
POST /api/v1/run/tasks
{
  "prompt": "Find the top 3 posts on Hacker News and return their titles and scores.",
  "data_extraction_schema": {
    "type": "array",
    "items": {
      "type": "object",
      "properties": {
        "title": {"type": "string"},
        "score": {"type": "integer"}
      }
    }
  }
}
```

**Response:**

```json
{
  "run_id": "tsk_v2_abc123",
  "status": "queued",
  "run_type": "task_v2",
  "output": null,
  "created_at": "2025-01-01T00:00:00Z",
  "modified_at": "2025-01-01T00:00:00Z"
}
```

---

### Get a run

```
GET /runs/{run_id}
```

Returns the current status and output of any run — task or workflow. The `run_id` starts with `tsk_` for tasks and `wr_` for workflow runs.

**Path parameters:**

| Parameter | Description |
|---|---|
| `run_id` | The task or workflow run ID |

**Response:**

```json
{
  "run_id": "tsk_v2_abc123",
  "status": "completed",
  "run_type": "task_v2",
  "output": [
    {"title": "Show HN: My project", "score": 342},
    {"title": "Ask HN: Best tools for X?", "score": 178}
  ],
  "recording_url": "https://...",
  "screenshot_urls": ["https://..."],
  "failure_reason": null,
  "step_count": 4,
  "started_at": "2025-01-01T00:00:05Z",
  "finished_at": "2025-01-01T00:00:42Z",
  "created_at": "2025-01-01T00:00:00Z",
  "modified_at": "2025-01-01T00:00:42Z"
}
```

**Run statuses:** `created`, `queued`, `running`, `completed`, `failed`, `terminated`, `timed_out`, `canceled`

A `completed` status means the task succeeded. `terminated` means the agent decided the task was impossible. `failed` means an internal error occurred.

---

### Cancel a run

```
POST /runs/{run_id}/cancel
```

Cancels an in-progress task or workflow run. Returns `204 No Content` on success.

---

### Get run artifacts

```
GET /runs/{run_id}/artifacts
```

Returns all artifacts produced during the run — screenshots, recordings, LLM responses, HAR files, etc.

**Query parameters:**

| Parameter | Type | Description |
|---|---|---|
| `artifact_type` | string (repeatable) | Filter by type: `recording`, `screenshot_action`, `screenshot_llm`, `llm_response`, `html_scrape`, `pdf`, `har`, `trace` |

**Response:** Array of `Artifact` objects, each with a `signed_url` valid for 24 hours.

---

### Get run timeline

```
GET /runs/{run_id}/timeline
```

Returns a structured timeline of events for a workflow run or task_v2 run — useful for debugging what happened at each stage.

---

### Retry run webhook

```
POST /runs/{run_id}/retry_webhook
```

Resends the webhook notification for a completed run. Useful if your webhook endpoint was down when the run finished.

---

## Workflows

### Create a workflow

```
POST /workflows
```

Creates a new workflow from a YAML or JSON definition. The workflow is not executed — use `POST /run/workflows` to run it.

**Request body:**

```json
{
  "yaml_definition": "<YAML string>",
  "json_definition": null
}
```

Submit either `yaml_definition` (a YAML string) or `json_definition` (a parsed JSON object). Block labels and parameter keys are automatically sanitized to valid identifiers on import.

**Response:** A `Workflow` object with `workflow_id` (starts with `wpid_`).

---

### Update a workflow

```
POST /workflows/{workflow_id}
```

Replaces the workflow definition. Creates a new version while preserving the permanent ID.

---

### Delete a workflow

```
POST /workflows/{workflow_id}/delete
```

Permanently deletes the workflow and all its versions.

---

### List workflows

```
GET /workflows
```

**Query parameters:**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `folder_id` | string | `null` | Filter by folder |
| `limit` | integer | `20` | Max results |
| `offset` | integer | `0` | Pagination offset |
| `sort` | string | `created_at` | Sort field |
| `order` | string | `desc` | Sort order: `asc` or `desc` |

---

### Run a workflow

```
POST /run/workflows
```

Starts a workflow execution and returns immediately with a `workflow_run_id`.

**Request body:**

| Field | Type | Default | Description |
|---|---|---|---|
| `workflow_id` | string | required | The workflow permanent ID (`wpid_...`) |
| `parameters` | object | `null` | Key-value pairs passed to workflow parameters |
| `proxy_location` | string/object | `RESIDENTIAL` | Proxy region |
| `webhook_url` | string | `null` | URL to POST results to on completion |
| `browser_session_id` | string | `null` | Reuse a persistent browser session |
| `totp_url` | string | `null` | URL for TOTP code polling |
| `ai_fallback` | boolean | `null` | Fall back to AI if script execution fails |
| `run_with` | string | `null` | Execution mode: `agent`, `code`, `code_v2` |

**Example:**

```json
POST /api/v1/run/workflows
{
  "workflow_id": "wpid_abc123",
  "parameters": {
    "target_url": "https://example.com",
    "max_items": 10
  }
}
```

**Response:**

```json
{
  "run_id": "wr_xyz789",
  "status": "created",
  "run_type": "workflow_run",
  "output": null,
  "created_at": "2025-01-01T00:00:00Z",
  "modified_at": "2025-01-01T00:00:00Z"
}
```

---

## Credentials

Credentials store secrets (passwords, credit cards, API keys) encrypted in the database. Raw secrets are never returned by the API — only non-sensitive metadata.

### Create a credential

```
POST /credentials
```

**Request body:**

| Field | Type | Description |
|---|---|---|
| `name` | string | Human-readable label |
| `credential_type` | string | `password`, `credit_card`, or `secret` |
| `credential` | object | Type-specific credential data (see below) |

**Password credential:**
```json
{
  "name": "Amazon Login",
  "credential_type": "password",
  "credential": {
    "username": "user@example.com",
    "password": "securepassword123",
    "totp": "JBSWY3DPEHPK3PXP",
    "totp_type": "authenticator"
  }
}
```

**Credit card credential:**
```json
{
  "name": "Company Card",
  "credential_type": "credit_card",
  "credential": {
    "card_number": "4111111111111111",
    "card_cvv": "123",
    "card_exp_month": "12",
    "card_exp_year": "2027",
    "card_brand": "visa",
    "card_holder_name": "Jane Smith"
  }
}
```

**Secret credential:**
```json
{
  "name": "API Token",
  "credential_type": "secret",
  "credential": {
    "secret_value": "sk-abc123xyz",
    "secret_label": "OpenAI Key"
  }
}
```

**Response:** A `CredentialResponse` with `credential_id` and non-sensitive fields only (username, last four digits, or secret label).

---

### List credentials

```
GET /credentials
```

Returns all credentials for the organization. Sensitive data is never included.

---

### Delete a credential

```
DELETE /credentials/{credential_id}
```

---

### Send a TOTP code

```
POST /credentials/totp
```

Forward a one-time password (from an email or SMS) to Skyvern so it can complete 2FA during a running task.

**Request body:**

| Field | Type | Description |
|---|---|---|
| `content` | string | The OTP message or code |
| `totp_identifier` | string | The email or phone the code was sent to |
| `task_id` | string | (optional) Associate with a specific task |
| `workflow_run_id` | string | (optional) Associate with a workflow run |

---

## Artifacts

### Get an artifact

```
GET /artifacts/{artifact_id}
```

Returns artifact metadata and a signed URL valid for 24 hours.

**Artifact types:**

| Type | Description |
|---|---|
| `recording` | Video recording of the browser session |
| `screenshot_action` | Screenshot taken after each action |
| `screenshot_llm` | Screenshot sent to the LLM at each step |
| `llm_response` | Raw LLM output for each step |
| `html_scrape` | HTML snapshot of the page |
| `har` | HTTP Archive (network requests) |
| `trace` | Playwright trace file |
| `pdf` | PDF generated by the print_page block |

---

## Authentication

All API keys belong to an organization. Every resource (tasks, workflows, credentials, artifacts) is scoped to that organization.

**Local development:** The local API key is stored in `.env` after running `skyvern quickstart`. You can find it in the Skyvern UI under Settings.

**Skyvern Cloud:** Generate an API key at `app.skyvern.com` under Settings → API Keys.

Pass the key in the `x-api-key` header on every request:

```bash
curl -X POST https://api.skyvern.com/api/v1/run/tasks \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Get the top Hacker News post"}'
```

---

## Error responses

All errors return a JSON body with a `detail` field:

```json
{"detail": "Task run not found tsk_abc123"}
```

| Status | Meaning |
|---|---|
| `400` | Bad request — invalid parameters |
| `401` | Missing or invalid API key |
| `404` | Resource not found |
| `422` | Validation error (invalid schema, YAML parse failure) |
| `429` | Rate limit exceeded |
| `500` | Internal server error |
