# API Reference

All requests go to `http://localhost:8000/api/v1` (or your deployed base URL). Every endpoint requires an `x-api-key` header. You can find your API key in the Skyvern UI under **Settings**.

```
x-api-key: <your-api-key>
Content-Type: application/json
```

Responses are JSON. Errors follow the shape `{"detail": "message"}` with an appropriate HTTP status code.

---

## Tasks

### Run a task

```
POST /run/tasks
```

Submits a browser-automation task and immediately begins execution. Returns as soon as the task is created; poll `GET /runs/{run_id}` for results.

**Request body**

| Field | Type | Required | Description |
|---|---|---|---|
| `prompt` | string | yes | Natural-language description of what to do on the page |
| `url` | string | no | Starting URL. Can be omitted if the agent should infer it from `prompt` |
| `engine` | string | no | Execution engine. Default: `skyvern_v2`. Options: `skyvern_v1`, `skyvern_v2`, `openai_cua`, `anthropic_cua` |
| `data_extraction_schema` | object/array/string | no | JSON schema describing the data you want extracted |
| `max_steps` | integer | no | Maximum number of agent steps. Overrides the server default |
| `webhook_url` | string | no | URL to POST results to when the task finishes |
| `proxy_location` | string/object | no | Proxy region (e.g., `"RESIDENTIAL"`) or a `{"country", "subdivision", "city"}` GeoTarget |
| `browser_session_id` | string | no | Reuse an existing browser session (preserves cookies, login state) |
| `error_code_mapping` | object | no | Map custom error codes to plain-language descriptions |
| `totp_identifier` | string | no | Identifier for a stored TOTP credential |
| `totp_url` | string | no | URL Skyvern should call to retrieve a one-time password |
| `extra_http_headers` | object | no | Extra HTTP headers the browser sends on every request |
| `title` | string | no | Human-readable label for this run |

```json
{
  "prompt": "Go to the checkout page and complete the purchase using the saved card",
  "url": "https://shop.example.com/cart",
  "data_extraction_schema": {
    "type": "object",
    "properties": {
      "order_number": { "type": "string" },
      "total": { "type": "number" }
    }
  },
  "webhook_url": "https://my-server.com/webhooks/skyvern",
  "max_steps": 20
}
```

**Response** `200 OK`

```json
{
  "run_id": "tsk_v2_abc123",
  "run_type": "task_v2",
  "status": "created",
  "output": null,
  "failure_reason": null,
  "created_at": "2025-01-15T10:00:00Z",
  "modified_at": "2025-01-15T10:00:00Z",
  "app_url": "https://app.skyvern.com/runs/tsk_v2_abc123"
}
```

---

### Get a run

```
GET /runs/{run_id}
```

Returns the current status and output of a task run or workflow run. Poll this endpoint until `status` is a terminal value.

**Path parameters**

| Parameter | Description |
|---|---|
| `run_id` | The `run_id` returned by `POST /run/tasks` or `POST /run/workflows`. Starts with `tsk_`, `tsk_v2_`, or `wr_` |

**Response** `200 OK`

```json
{
  "run_id": "tsk_v2_abc123",
  "run_type": "task_v2",
  "status": "completed",
  "output": {
    "order_number": "ORD-98765",
    "total": 49.99
  },
  "failure_reason": null,
  "screenshot_urls": ["https://..."],
  "recording_url": "https://...",
  "step_count": 7,
  "created_at": "2025-01-15T10:00:00Z",
  "modified_at": "2025-01-15T10:05:12Z",
  "started_at": "2025-01-15T10:00:02Z",
  "finished_at": "2025-01-15T10:05:12Z",
  "app_url": "https://app.skyvern.com/runs/tsk_v2_abc123"
}
```

**Terminal status values**

| Status | Meaning |
|---|---|
| `completed` | Task finished successfully. Check `output` for results |
| `failed` | Task failed. See `failure_reason` |
| `terminated` | Stopped because a `terminate_criterion` was met |
| `timed_out` | Exceeded the maximum allowed runtime |
| `canceled` | Canceled via `POST /runs/{run_id}/cancel` |

---

### Cancel a run

```
POST /runs/{run_id}/cancel
```

Stops a running task or workflow. Returns `204 No Content` on success.

---

## Workflows

### Run a workflow

```
POST /run/workflows
```

Starts an execution of an existing workflow definition. Returns a `run_id` you use to track the run with `GET /runs/{run_id}`.

**Request body**

| Field | Type | Required | Description |
|---|---|---|---|
| `workflow_id` | string | yes | The workflow's permanent ID (starts with `wpid_`) |
| `parameters` | object | no | Input parameter values for this run |
| `webhook_url` | string | no | URL to POST final results to |
| `proxy_location` | string/object | no | Proxy region or GeoTarget |
| `browser_session_id` | string | no | Reuse an existing browser session |
| `title` | string | no | Human-readable label for this run |

```json
{
  "workflow_id": "wpid_xyz789",
  "parameters": {
    "product_url": "https://store.example.com/item/42",
    "quantity": 2
  },
  "webhook_url": "https://my-server.com/webhooks/skyvern"
}
```

**Response** `200 OK`

```json
{
  "run_id": "wr_def456",
  "run_type": "workflow_run",
  "status": "created",
  "output": null,
  "created_at": "2025-01-15T11:00:00Z",
  "modified_at": "2025-01-15T11:00:00Z",
  "app_url": "https://app.skyvern.com/runs/wr_def456"
}
```

### Create a workflow

```
POST /workflows
```

Creates a new workflow from a YAML definition. See the [Workflows](workflows.md) guide for the full definition format.

**Request** — YAML body (`Content-Type: application/x-yaml`)

```yaml
title: "Purchase product"
description: "Navigates to a product URL and adds it to the cart"
parameters:
  - parameter_type: workflow
    key: product_url
    workflow_parameter_type: string

blocks:
  - block_type: task
    label: add_to_cart
    url: "{{ parameters.product_url }}"
    navigation_goal: "Add the item to the cart"
```

**Response** `200 OK` — Returns the created workflow object including its `workflow_id`.

### Get workflows

```
GET /workflows
```

Lists all workflows for your organization. Supports `page` and `page_size` query parameters.

### Get a workflow

```
GET /workflows/{workflow_id}
```

Returns a workflow definition by its permanent ID.

### Update a workflow

```
PUT /workflows/{workflow_id}
```

Replaces a workflow definition. Creates a new version; existing runs are unaffected.

### Delete a workflow

```
DELETE /workflows/{workflow_id}
```

Permanently removes a workflow. Running instances continue to completion.

---

## Browser Sessions

Browser sessions let you keep a browser alive across multiple task runs, preserving login state, cookies, and page context.

### Create a browser session

```
POST /browser_sessions
```

**Request body**

| Field | Type | Default | Description |
|---|---|---|---|
| `timeout` | integer | `60` | Session lifetime in minutes (5–1440) |
| `proxy_location` | string | `null` | Proxy region |
| `browser_profile_id` | string | `null` | Load a saved browser profile |
| `browser_type` | string | `null` | Browser variant |

```json
{
  "timeout": 120,
  "proxy_location": "RESIDENTIAL"
}
```

**Response** `200 OK`

```json
{
  "browser_session_id": "pbs_ghi012",
  "status": "running",
  "created_at": "2025-01-15T12:00:00Z"
}
```

Pass the `browser_session_id` to any `POST /run/tasks` or `POST /run/workflows` call to reuse this session.

### Get a browser session

```
GET /browser_sessions/{browser_session_id}
```

### List browser sessions

```
GET /browser_sessions
```

### Close a browser session

```
POST /browser_sessions/{browser_session_id}/close
```

Terminates the session and releases the underlying browser process.

---

## Credentials

Credentials store sensitive values (passwords, credit cards, secrets) encrypted at rest. The API never returns raw credential data — only non-sensitive metadata.

### Create a credential

```
POST /credentials
```

**Request body**

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | yes | Human-readable label (e.g., `"Amazon Login"`) |
| `credential_type` | string | yes | `"password"`, `"credit_card"`, or `"secret"` |
| `credential` | object | yes | The credential data (see below) |

For `credential_type: "password"`:
```json
{
  "name": "Amazon Login",
  "credential_type": "password",
  "credential": {
    "username": "user@example.com",
    "password": "s3cur3p@ss"
  }
}
```

For `credential_type: "credit_card"`:
```json
{
  "name": "Company Card",
  "credential_type": "credit_card",
  "credential": {
    "card_number": "4111111111111111",
    "card_cvv": "123",
    "card_exp_month": "12",
    "card_exp_year": "2027",
    "card_holder_name": "Jane Smith"
  }
}
```

**Response** `200 OK`

```json
{
  "credential_id": "cred_jkl345",
  "credential_type": "password",
  "name": "Amazon Login",
  "created_at": "2025-01-15T09:00:00Z"
}
```

### List credentials

```
GET /credentials
```

Returns metadata only — never raw credential values.

### Delete a credential

```
DELETE /credentials/{credential_id}
```

---

## Webhooks

When you provide a `webhook_url` on a task or workflow run, Skyvern sends a `POST` request to that URL when the run finishes. The request body matches the shape of `GET /runs/{run_id}`.

Skyvern signs webhook requests with an HMAC-SHA256 signature in the `x-skyvern-signature` header. To verify:

```python
import hmac, hashlib

def verify_webhook(secret: str, payload: bytes, signature: str) -> bool:
    expected = hmac.new(secret.encode(), payload, hashlib.sha256).hexdigest()
    return hmac.compare_digest(expected, signature)
```

Set your webhook secret via the `SKYVERN_WEBHOOK_SIGNING_KEY` environment variable on the server.

---

## WebSocket streaming

For real-time updates during task execution, connect to the WebSocket endpoint:

```
ws://localhost:8000/api/v1/ws
```

After connecting, subscribe to a run:

```json
{ "action": "subscribe", "run_id": "tsk_v2_abc123" }
```

You'll receive events as the agent takes each step:

```json
{
  "event": "step_completed",
  "run_id": "tsk_v2_abc123",
  "step": 3,
  "screenshot_url": "https://...",
  "action": "click",
  "status": "running"
}
```

---

## Error responses

| HTTP Status | When it occurs |
|---|---|
| `400 Bad Request` | Invalid request body (bad URL, unsupported engine, etc.) |
| `401 Unauthorized` | Missing or invalid `x-api-key` |
| `403 Forbidden` | The API key exists but lacks permission |
| `404 Not Found` | Run, workflow, or credential ID does not exist |
| `422 Unprocessable Entity` | Pydantic validation failure — check field types |
| `429 Too Many Requests` | Rate limit exceeded |
| `500 Internal Server Error` | Unexpected server failure |

Error bodies look like:

```json
{ "detail": "Task run not found tsk_v2_abc123" }
```

For validation errors, `detail` is an array:

```json
{
  "detail": [
    { "loc": ["body", "url"], "msg": "Invalid URL: not-a-url", "type": "value_error" }
  ]
}
```
