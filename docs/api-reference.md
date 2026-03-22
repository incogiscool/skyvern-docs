# API Reference

Skyvern exposes a REST API and a Python SDK (`skyvern`) that mirror each other one-to-one. Every endpoint shown here is available at `https://api.skyvern.com` (Skyvern Cloud) or `http://localhost:8000` (self-hosted). All paths are prefixed with `/v1`.

---

## Authentication

Every request must include your API key in the `x-api-key` header.

```http
x-api-key: YOUR_API_KEY
```

You can find your API key in the Skyvern dashboard under **Settings**. The SDK picks it up automatically when you instantiate the client:

```python
from skyvern import Skyvern

client = Skyvern(api_key="YOUR_API_KEY")
```

For self-hosted deployments, pass `base_url` instead of relying on the default Cloud environment:

```python
from skyvern import Skyvern

client = Skyvern(
    api_key="YOUR_API_KEY",
    base_url="http://localhost:8000",
)
```

Requests without a valid key return `403 Forbidden`.

---

## Run statuses

Both task runs and workflow runs share the same status lifecycle:

| Status | Meaning |
|---|---|
| `created` | Run accepted, not yet queued |
| `queued` | Waiting for a browser worker |
| `running` | Browser is active |
| `completed` | Finished successfully |
| `failed` | Finished with an error |
| `terminated` | Stopped by the system |
| `timed_out` | Exceeded the step or time budget |
| `canceled` | Canceled via the API |

---

## Tasks

A task is a single browser automation goal. You give Skyvern a `prompt` and an optional `url`, and it figures out the rest.

### Run a task

```http
POST /v1/run/tasks
```

Starts a new task and returns immediately. The task runs asynchronously; poll `GET /v1/runs/{run_id}` to check progress, or provide a `webhook_url` to receive a callback when it finishes.

**Request body**

| Field | Type | Default | Description |
|---|---|---|---|
| `prompt` | `string` | **required** | The goal for Skyvern to accomplish, in plain language. |
| `url` | `string` | `null` | Starting URL. If omitted, Skyvern infers an appropriate URL from the prompt. |
| `engine` | `string` | `"skyvern-2.0"` | Agent engine. See [engines](#engines) below. |
| `title` | `string` | `null` | Human-readable label for this run. |
| `proxy_location` | `string` or `GeoTarget` | `"RESIDENTIAL"` | Proxy region. See [proxy locations](#proxy-locations). |
| `data_extraction_schema` | `object` or `array` or `string` | `null` | JSON Schema describing the structured output you want. When supplied, `output` in the response will conform to this shape. |
| `error_code_mapping` | `object` | `null` | Map error descriptions to short codes. For example, `{"Invalid credentials": "LOGIN_FAILED"}`. |
| `max_steps` | `integer` | `null` | Hard cap on agent steps. The task fails if this is exceeded. Charged per step. |
| `webhook_url` | `string` | `null` | URL to POST a completion payload to when the run finishes. |
| `totp_identifier` | `string` | `null` | Identifier used when pushing a 2FA code to Skyvern (option 3 of TOTP flow). |
| `totp_url` | `string` | `null` | URL Skyvern polls to fetch 2FA codes (option 2 of TOTP flow). |
| `browser_session_id` | `string` | `null` | Attach this run to an existing persistent browser session. Starts from that session's current browser state. |
| `extra_http_headers` | `object` | `null` | Additional HTTP headers injected into every browser request during the run. |
| `max_screenshot_scrolls` | `integer` | `null` | Maximum scroll depth for post-action screenshots. `0` or `null` captures only the viewport. |

**Python SDK**

```python
response = client.run_task(
    prompt="Search for 'Skyvern' on Google and return the first three result titles.",
    url="https://google.com",
    engine="skyvern-2.0",
    data_extraction_schema={
        "type": "object",
        "properties": {
            "results": {
                "type": "array",
                "items": {"type": "string"}
            }
        }
    },
    max_steps=10,
    webhook_url="https://your-server.com/webhook",
)
print(response.run_id)   # e.g. "tsk_01j..."
print(response.status)   # "created" or "running"
```

**curl**

```bash
curl -X POST https://api.skyvern.com/v1/run/tasks \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Search for Skyvern on Google and return the first three result titles.",
    "url": "https://google.com",
    "max_steps": 10
  }'
```

**Response**

```json
{
  "run_id": "tsk_01j9abc123",
  "status": "created",
  "output": null,
  "failure_reason": null,
  "recording_url": null,
  "screenshot_urls": null,
  "created_at": "2025-01-15T10:30:00Z",
  "modified_at": "2025-01-15T10:30:00Z",
  "app_url": "https://app.skyvern.com/runs/tsk_01j9abc123"
}
```

The `run_id` starting with `tsk_` identifies this as a task run. Use it to poll for results.

---

#### Engines

The `engine` field controls which agent model drives the run.

| Value | When to use |
|---|---|
| `skyvern-2.0` | Default. Best for complex, multi-step tasks and anything involving dynamic pages. |
| `skyvern-1.0` | Simple single-page tasks: form filling, basic searches. Faster and cheaper. |
| `openai-cua` | OpenAI's Computer Use Agent model. |
| `anthropic-cua` | Anthropic's Claude Sonnet 3.7 with computer use. |
| `ui-tars` | UI-TARS model. |

---

## Getting run results

### Get a run

```http
GET /v1/runs/{run_id}
```

Returns the current state of any run, whether a task or a workflow run. The `run_type` field in the response discriminates between them.

| Path parameter | Description |
|---|---|
| `run_id` | Task ID (`tsk_...`), task v2 ID, or workflow run ID (`wr_...`). |

**Python SDK**

```python
run = client.get_run(run_id="tsk_01j9abc123")
print(run.status)        # "completed"
print(run.output)        # your extracted data
print(run.recording_url) # link to browser recording
```

**Response fields**

| Field | Type | Description |
|---|---|---|
| `run_id` | `string` | Unique identifier for this run. |
| `run_type` | `string` | `task_v1`, `task_v2`, `workflow_run`, `openai_cua`, `anthropic_cua`, or `ui_tars`. |
| `status` | `string` | Current run status. |
| `output` | `object` or `null` | Extracted data. Shape matches `data_extraction_schema` if one was provided. |
| `failure_reason` | `string` or `null` | Human-readable explanation when status is `failed` or `terminated`. |
| `recording_url` | `string` or `null` | URL to the screen recording of this run. |
| `screenshot_urls` | `array` or `null` | Last N screenshots in reverse chronological order (most recent first). |
| `downloaded_files` | `array` or `null` | Files downloaded during the run. |
| `errors` | `array` or `null` | Error objects. Each has an `error_code` field when `error_code_mapping` was used. |
| `step_count` | `integer` or `null` | Total agent steps executed. |
| `started_at` | `datetime` or `null` | When execution began. |
| `finished_at` | `datetime` or `null` | When execution ended. |
| `app_url` | `string` or `null` | Link to this run in the Skyvern UI. |
| `run_request` | `object` or `null` | The original request parameters. |

A common polling pattern while waiting for a run to finish:

```python
import time

def wait_for_run(client, run_id, poll_interval=5):
    terminal = {"completed", "failed", "terminated", "timed_out", "canceled"}
    while True:
        run = client.get_run(run_id=run_id)
        if run.status in terminal:
            return run
        time.sleep(poll_interval)

run = wait_for_run(client, "tsk_01j9abc123")
print(run.output)
```

For production use, webhooks are more efficient than polling. See [Webhooks](#webhooks).

---

### Cancel a run

```http
POST /v1/runs/{run_id}/cancel
```

Cancels a run that is `created`, `queued`, or `running`. Works for both tasks and workflow runs.

```python
client.cancel_run(run_id="tsk_01j9abc123")
```

---

### Get run artifacts

```http
GET /v1/runs/{run_id}/artifacts
```

Returns the debug artifacts produced during a run: screenshots, HTML snapshots, LLM prompts, recordings, and more. These are useful for diagnosing unexpected behavior.

| Query parameter | Type | Description |
|---|---|---|
| `artifact_type` | `string` (repeatable) | Filter to specific artifact types. |

Available `artifact_type` values: `recording`, `screenshot`, `screenshot_llm`, `screenshot_action`, `screenshot_final`, `html`, `llm_prompt`, `llm_request`, `llm_response`, `visible_elements_tree`, `har`, `trace`, `pdf`, `browser_console_log`.

```python
artifacts = client.get_run_artifacts(run_id="tsk_01j9abc123")
for a in artifacts:
    print(a.artifact_type, a.signed_url)
```

Each artifact has:

| Field | Type | Description |
|---|---|---|
| `artifact_id` | `string` | Unique identifier. |
| `artifact_type` | `string` | Type label from the list above. |
| `uri` | `string` | Internal storage URI. |
| `signed_url` | `string` or `null` | Time-limited public download URL. |
| `created_at` | `string` | ISO 8601 timestamp. |

---

### Get run timeline

```http
GET /v1/runs/{run_id}/timeline
```

Returns a step-by-step timeline of what happened during a workflow run or a `skyvern-2.0` task run. Useful for understanding which blocks executed and in what order.

```python
timeline = client.get_run_timeline(run_id="wr_01j9abc123")
for step in timeline:
    print(step.block_type, step.status)
```

---

### Retry a webhook

If a webhook delivery failed, you can trigger a retry without re-running the task:

```http
POST /v1/runs/{run_id}/retry_webhook
```

```python
client.retry_run_webhook(run_id="tsk_01j9abc123")

# Or deliver to a different URL this time:
from skyvern import RetryRunWebhookRequest
client.retry_run_webhook(
    run_id="tsk_01j9abc123",
    request=RetryRunWebhookRequest(webhook_url="https://other-server.com/webhook"),
)
```

---

## Workflows

Workflows are multi-step automations defined as a sequence of blocks. They accept typed parameters, support loops and conditionals, and are versioned with a permanent ID.

### Run a workflow

```http
POST /v1/run/workflows
```

Starts a workflow run. The `workflow_id` must start with `wpid_`.

**Request body**

| Field | Type | Default | Description |
|---|---|---|---|
| `workflow_id` | `string` | **required** | The permanent workflow ID (`wpid_...`). |
| `parameters` | `object` | `null` | Key-value pairs matching the workflow's declared parameters. |
| `title` | `string` | `null` | Label for this specific run. |
| `proxy_location` | `string` or `GeoTarget` | `"RESIDENTIAL"` | Proxy region. |
| `webhook_url` | `string` | `null` | Completion webhook URL. |
| `totp_identifier` | `string` | `null` | 2FA push identifier. |
| `totp_url` | `string` | `null` | 2FA pull URL. |
| `browser_session_id` | `string` | `null` | Persistent browser session to attach. |
| `browser_profile_id` | `string` | `null` | Browser profile to load (restores cookies and localStorage). |
| `max_steps_override` | `integer` | `null` | Override the per-task step limit for this run. |
| `extra_http_headers` | `object` | `null` | Additional browser HTTP headers. |

```python
run = client.run_workflow(
    workflow_id="wpid_01jabc",
    parameters={
        "company_name": "Acme Corp",
        "max_records": 50,
    },
    webhook_url="https://your-server.com/webhook",
)
print(run.run_id)  # "wr_01j..."
```

The run ID for workflows starts with `wr_`. Pass it to `get_run` or `cancel_run` just like a task run ID.

---

### List workflows

```http
GET /v1/workflows
```

Returns all workflows for your organization, latest version only.

| Query parameter | Type | Description |
|---|---|---|
| `page` | `integer` | Page number (1-indexed). |
| `page_size` | `integer` | Results per page. |
| `search_key` | `string` | Case-insensitive search across workflow title, folder name, and parameter metadata. |
| `folder_id` | `string` | Filter to a specific folder. |
| `status` | `string` (repeatable) | Filter by workflow status. |
| `only_workflows` | `boolean` | Exclude saved tasks (tasks promoted to workflows). |
| `only_templates` | `boolean` | Show only template workflows. |

```python
workflows = client.get_workflows(page=1, page_size=20)
for wf in workflows:
    print(wf.workflow_id, wf.title)
```

---

### Get a workflow

```http
GET /v1/workflows/{workflow_id}
```

Returns a workflow's definition and metadata.

```python
wf = client.get_workflow(workflow_id="wpid_01jabc")
print(wf.title)
print(wf.workflow_definition)  # block definitions, parameters
```

---

### Create a workflow

```http
POST /v1/workflows
```

Creates a new workflow from a JSON or YAML definition.

```python
workflow = client.create_workflow(
    yaml_definition="""
title: "My Workflow"
description: "Automates a multi-step process"
parameters:
  - key: target_url
    parameter_type: workflow
    data_type: str
    default_value: "https://example.com"
blocks:
  - block_type: navigation
    label: "Go to the site"
    url: "{{target_url}}"
    navigation_goal: "Navigate to the pricing page"
"""
)
print(workflow.workflow_id)  # "wpid_..."
```

---

### List workflow runs

```http
GET /v1/workflow_runs
```

Lists runs across all workflows for your organization.

| Query parameter | Type | Description |
|---|---|---|
| `page` | `integer` | Page number. |
| `page_size` | `integer` | Results per page. |
| `status` | `string` (repeatable) | Filter by run status. |
| `search_key` | `string` | Search across run ID, parameter keys, parameter values, and headers. |
| `error_code` | `string` | Exact match on an error code in any task within the run. |

```python
# Find all failed runs that encountered a login error
runs = client.get_workflow_runs(
    status="failed",
    error_code="LOGIN_FAILED",
    page_size=50,
)
```

---

## Credentials

Credentials let Skyvern log into sites on your behalf without you exposing passwords in task prompts. They are stored encrypted in Skyvern's vault.

**Security note:** The API never returns raw credential data (passwords, card numbers, CVVs, TOTP secrets). Only non-sensitive metadata is returned.

### Create a credential

```http
POST /v1/credentials
```

| Field | Type | Description |
|---|---|---|
| `name` | `string` | Label for this credential (e.g., `"Amazon Login"`). |
| `credential_type` | `string` | `"password"`, `"credit_card"`, or `"secret"`. |
| `credential` | `object` | Credential data. Shape depends on `credential_type`. |

**Password credential**

```python
from skyvern import NonEmptyPasswordCredential

cred = client.create_credential(
    name="My Amazon Login",
    credential_type="password",
    credential=NonEmptyPasswordCredential(
        username="user@example.com",
        password="securepassword123",
        # Optional: include a TOTP secret if this account uses an authenticator app
        totp="JBSWY3DPEHPK3PXP",
        totp_type="authenticator",
    ),
)
print(cred.credential_id)  # "cred_..."
```

**Credit card credential**

```python
from skyvern import NonEmptyCreditCardCredential

cred = client.create_credential(
    name="Corporate Visa",
    credential_type="credit_card",
    credential=NonEmptyCreditCardCredential(
        card_number="4111111111111111",
        card_cvv="123",
        card_exp_month="12",
        card_exp_year="2027",
        card_brand="visa",
        card_holder_name="Jane Doe",
    ),
)
```

To use a credential in a task, reference it in your prompt by name or configure the workflow with a `CREDENTIAL` parameter type.

---

### List credentials

```http
GET /v1/credentials
```

```python
creds = client.get_credentials()
for c in creds:
    print(c.credential_id, c.name, c.credential_type)
```

Each credential in the list has `credential_id`, `name`, `credential_type`, and safe metadata (username, last four digits, etc.) but no secrets.

---

### Delete a credential

```http
DELETE /v1/credentials/{credential_id}
```

```python
client.delete_credential(credential_id="cred_01jabc")
```

---

## 2FA / TOTP

When a site requires a time-based one-time password (TOTP) or a code sent by email/SMS, you need to route that code to Skyvern during the run.

There are three options:

**Option 1: Stored authenticator secret** Supply the TOTP base32 secret when creating a credential (the `totp` field). Skyvern generates codes automatically. No extra work at runtime.

**Option 2: Pull from your endpoint** Set `totp_url` when starting the run. Skyvern polls that URL for codes as needed. Your endpoint should return `{"code": "123456"}`.

**Option 3: Push from your system** Set `totp_identifier` when starting the run (typically an email address or phone number). When you receive the 2FA message, call `send_totp_code` to deliver it:

```http
POST /v1/totp
```

```python
# Called from your email/SMS processing pipeline when a 2FA message arrives
client.send_totp_code(
    totp_identifier="user@example.com",
    content="Your verification code is 847291. Valid for 10 minutes.",
    # Optional: pin to a specific run
    workflow_run_id="wr_01jabc",
)
```

Skyvern extracts the numeric code from `content` automatically, so you can pass the full email or SMS body.

| Field | Type | Description |
|---|---|---|
| `totp_identifier` | `string` | **required.** The email, phone number, or custom identifier that links this code to a pending run. |
| `content` | `string` | **required.** Full message body. Skyvern parses out the OTP. |
| `task_id` | `string` | Pin to a specific task run. |
| `workflow_run_id` | `string` | Pin to a specific workflow run. |
| `expired_at` | `datetime` | When this code expires. |
| `type` | `string` | `"totp"` or `"magic_link"`. Force extraction of one type when the message contains both. |

---

## Browser Sessions

A browser session is a long-lived browser container that persists its state between runs. Use one when you need a series of tasks to share cookies, localStorage, or to avoid repeated logins.

### Create a session

```http
POST /v1/browser_sessions
```

| Field | Type | Default | Description |
|---|---|---|---|
| `timeout` | `integer` | `60` | Minutes until the session auto-expires (5–1440). |
| `proxy_location` | `string` | `"RESIDENTIAL"` | Proxy region for the session's browser. |
| `browser_type` | `string` | `null` | `"chrome"` or `"msedge"`. |
| `browser_profile_id` | `string` | `null` | Profile ID (`bp_...`) to load into the session. |
| `extensions` | `array` | `null` | Browser extensions to install: `"ad-blocker"`, `"captcha-solver"`. |

```python
session = client.create_browser_session(
    timeout=120,
    proxy_location="RESIDENTIAL_GB",
)
session_id = session.browser_session_id  # "pbs_..."
```

Once you have a session ID, pass `browser_session_id` to any `run_task` or `run_workflow` call to attach that run to the session. The browser continues from wherever the previous run left off.

```python
# Log in once
login_run = client.run_task(
    prompt="Log in to example.com with username admin@example.com and password secret123",
    url="https://example.com/login",
    browser_session_id=session_id,
)
wait_for_run(client, login_run.run_id)

# Now run a task that requires an authenticated session
data_run = client.run_task(
    prompt="Download the Q3 sales report from the Reports page.",
    browser_session_id=session_id,
)
```

---

### Get a session

```http
GET /v1/browser_sessions/{browser_session_id}
```

```python
session = client.get_browser_session(browser_session_id="pbs_01jabc")
print(session.status)          # "running", "completed", etc.
print(session.browser_address) # CDP WebSocket URL for direct connection
```

The `browser_address` field is a Chrome DevTools Protocol (CDP) endpoint. You can connect external tooling (Playwright, Puppeteer) directly to a running Skyvern browser session via this address.

---

### List active sessions

```http
GET /v1/browser_sessions
```

```python
sessions = client.get_browser_sessions()
```

---

### Close a session

```http
POST /v1/browser_sessions/{browser_session_id}/close
```

Closed sessions cannot be reused. Sessions also close automatically when `timeout` is reached.

```python
client.close_browser_session(browser_session_id="pbs_01jabc")
```

---

## Proxy Locations

The `proxy_location` field on task and workflow runs routes browser traffic through a residential proxy in the chosen region.

| Value | Region |
|---|---|
| `RESIDENTIAL` | United States (default) |
| `RESIDENTIAL_GB` | United Kingdom |
| `RESIDENTIAL_DE` | Germany |
| `RESIDENTIAL_FR` | France |
| `RESIDENTIAL_ES` | Spain |
| `RESIDENTIAL_IE` | Ireland |
| `RESIDENTIAL_IN` | India |
| `RESIDENTIAL_JP` | Japan |
| `RESIDENTIAL_AU` | Australia |
| `RESIDENTIAL_CA` | Canada |
| `RESIDENTIAL_BR` | Brazil |
| `RESIDENTIAL_MX` | Mexico |
| `RESIDENTIAL_IT` | Italy |
| `RESIDENTIAL_NL` | Netherlands |
| `RESIDENTIAL_KR` | South Korea |
| `RESIDENTIAL_PH` | Philippines |
| `RESIDENTIAL_TR` | Turkey |
| `RESIDENTIAL_ZA` | South Africa |
| `RESIDENTIAL_AR` | Argentina |
| `RESIDENTIAL_NZ` | New Zealand |
| `RESIDENTIAL_ISP` | ISP proxy (US) |
| `NONE` | No proxy |

For city or state-level targeting, pass a `GeoTarget` object instead of a string:

```python
run = client.run_task(
    prompt="Check local pricing on this site.",
    url="https://example.com",
    proxy_location={"country": "US", "subdivision": "CA", "city": "San Francisco"},
)
```

`GeoTarget` fields: `country` (ISO 3166-1 alpha-2, required), `subdivision` (ISO 3166-2 code, optional), `city` (English GeoNames name, optional).

---

## Webhooks

Set `webhook_url` on any run to receive a POST request when the run finishes. The payload is the same shape as the `GET /v1/runs/{run_id}` response.

### Verifying webhook signatures

Every webhook delivery includes two headers you should validate before processing the payload:

| Header | Description |
|---|---|
| `x-skyvern-signature` | HMAC-SHA256 of the raw request body, keyed with your API key. |
| `x-skyvern-timestamp` | Unix timestamp (seconds) of when the webhook was sent. |

To verify:

```python
import hashlib
import hmac
import json

def verify_skyvern_webhook(
    raw_body: bytes,
    signature_header: str,
    api_key: str,
) -> bool:
    expected = hmac.new(
        api_key.encode("utf-8"),
        msg=raw_body,
        digestmod=hashlib.sha256,
    ).hexdigest()
    return hmac.compare_digest(expected, signature_header)
```

Reject any request where `verify_skyvern_webhook` returns `False`, or where the timestamp is more than a few minutes old (to guard against replay attacks).

### Example webhook payload

```json
{
  "run_id": "tsk_01j9abc123",
  "run_type": "task_v2",
  "status": "completed",
  "output": {
    "results": ["Title One", "Title Two", "Title Three"]
  },
  "failure_reason": null,
  "recording_url": "https://storage.skyvern.com/recordings/tsk_01j9abc123.mp4",
  "step_count": 7,
  "started_at": "2025-01-15T10:30:05Z",
  "finished_at": "2025-01-15T10:31:42Z"
}
```

---

## Async Python SDK

All methods have an async equivalent. Use `AsyncSkyvern` for non-blocking usage:

```python
import asyncio
from skyvern import AsyncSkyvern

client = AsyncSkyvern(api_key="YOUR_API_KEY")

async def main():
    run = await client.run_task(
        prompt="Find the current ETH price on CoinGecko.",
        url="https://coingecko.com",
        data_extraction_schema={
            "type": "object",
            "properties": {"price_usd": {"type": "number"}}
        },
    )

    # Poll until done
    terminal = {"completed", "failed", "terminated", "timed_out", "canceled"}
    while run.status not in terminal:
        await asyncio.sleep(3)
        run = await client.get_run(run_id=run.run_id)

    print(run.output)

asyncio.run(main())
```

`AsyncSkyvern` accepts the same parameters as `Skyvern` and exposes the same methods. All async methods are prefixed with `await`.

---

## Error responses

The API returns standard HTTP status codes. Error bodies follow this shape:

```json
{
  "detail": "Task run not found tsk_01j9abc123"
}
```

| Status | Meaning |
|---|---|
| `400` | Invalid request parameters. |
| `403` | Missing or invalid API key. |
| `404` | Resource not found. |
| `422` | Validation error. `detail` contains per-field messages. |
| `500` | Server error. Retry after a short delay. |

Validation errors include a breakdown of which fields failed:

```json
{
  "detail": "prompt: Field required; max_steps: Input should be greater than 0"
}
```
