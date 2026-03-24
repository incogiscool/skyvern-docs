# API Reference

Skyvern's REST API lives at `https://api.skyvern.com/v1`. Every request must include your API key in the `x-api-key` header. You can find your key at [app.skyvern.com/settings](https://app.skyvern.com/settings).

```http
x-api-key: YOUR_API_KEY
```

If you're running Skyvern self-hosted, replace `https://api.skyvern.com` with your server's address (default: `http://localhost:8000`).

The API also has Python and TypeScript SDKs that mirror the REST surface:

**Python**
```bash
pip install skyvern
```
```python
from skyvern import Skyvern
skyvern = Skyvern(api_key="YOUR_API_KEY")
```

**TypeScript**
```bash
npm install @skyvern/client
```
```typescript
import { SkyvernClient } from "@skyvern/client";
const skyvern = new SkyvernClient({ apiKey: "YOUR_API_KEY" });
```

---

## Contents

- [Runs](#runs) — start tasks and workflows, poll for status, retrieve artifacts
- [Workflows](#workflows) — create, update, and manage reusable automation definitions
- [Credentials](#credentials) — store logins, credit cards, and secrets in Skyvern's vault
- [Browser Sessions](#browser-sessions) — keep a browser alive across multiple runs
- [Proxy Locations](#proxy-locations) — route browser traffic through geographic proxies
- [Errors](#errors) — HTTP status codes and failure semantics

---

## Runs

A **run** is the core unit of work in Skyvern. Submitting a task or triggering a workflow both produce a run object, identified by a `run_id` you poll for status. Task run IDs start with `tsk_` (v1) or `tsk_v2_` (v2). Workflow run IDs start with `wr_`. All run types share the same [Get a run](#get-a-run), [Cancel a run](#cancel-a-run), and [Get run artifacts](#get-run-artifacts) endpoints.

### Run a task

`POST /v1/run/tasks`

Describe what you want the browser to do. Skyvern figures out how to do it.

```python
from skyvern import Skyvern

skyvern = Skyvern(api_key="YOUR_API_KEY")
run = await skyvern.run_task(
    prompt="Find the top 3 posts on Hacker News.",
    url="https://news.ycombinator.com",
)
print(run.run_id)   # tsk_v2_abc123
print(run.status)   # created | running | completed | failed
```

```typescript
import { SkyvernClient } from "@skyvern/client";

const skyvern = new SkyvernClient({ apiKey: "YOUR_API_KEY" });
const run = await skyvern.runTask({
    body: {
        prompt: "Find the top 3 posts on Hacker News.",
        url: "https://news.ycombinator.com",
    }
});
```

**Request body — core**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `prompt` | `string` | required | The goal or task description for Skyvern to accomplish. |
| `url` | `string \| null` | `null` | Starting URL. If omitted, Skyvern infers one from the prompt. |
| `engine` | `string` | `"skyvern-2.0"` | Which agent to use. See [engines](#engines) below. |
| `title` | `string \| null` | `null` | Human-readable label for this run. |
| `max_steps` | `integer \| null` | `null` | Hard cap on steps. The run fails if this is exceeded. Each step costs credits, so set this to a reasonable value. |
| `data_extraction_schema` | `object \| array \| string \| null` | `null` | JSON Schema describing the structure of data you want extracted. Providing a schema makes the output more consistent. |
| `error_code_mapping` | `object \| null` | `null` | Map your own error codes to conditions. Example: `{"login_failed": "The credentials are incorrect"}`. |
| `webhook_url` | `string \| null` | `null` | URL to POST a completion notification to when the run finishes. |
| `proxy_location` | `string \| object \| null` | `"RESIDENTIAL"` | Geographic proxy to route browser traffic through. See [proxy locations](#proxy-locations). |

**Request body — 2FA / TOTP**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `totp_identifier` | `string \| null` | `null` | Identifier used to match incoming TOTP/2FA codes pushed via the [Send TOTP code](#send-totp-code) endpoint. |
| `totp_url` | `string \| null` | `null` | URL Skyvern polls to fetch TOTP/2FA codes. |

**Request body — advanced**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `browser_session_id` | `string \| null` | `null` | Run inside an existing persistent browser session (continues from the current screen state). |
| `publish_workflow` | `boolean` | `false` | When `true` with `skyvern-2.0`, saves a successful task as a reusable workflow. |
| `extra_http_headers` | `object \| null` | `null` | Additional HTTP headers injected into all browser requests during the run. |
| `browser_address` | `string \| null` | `null` | CDP address of a custom browser to connect to (e.g. `http://127.0.0.1:9222`). |
| `model` | `object \| null` | `null` | Override the LLM model. Example: `{"name": "gpt-4.1"}`. |

#### Engines

The `engine` field controls which agent handles the task. Use `skyvern-2.0` unless you have a specific reason to switch — it's the most capable and the one Skyvern actively develops.

| Engine | Best for | Run ID prefix |
|---|---|---|
| `skyvern-2.0` | Complex, multi-step tasks; default choice | `tsk_v2_` |
| `skyvern-1.0` | Simple, single-page tasks like form filling | `tsk_` |
| `openai-cua` | Tasks where you want OpenAI's Computer Use Agent model | `tsk_v2_` |
| `anthropic-cua` | Tasks where you want Anthropic Claude Sonnet 3.7 with computer use | `tsk_v2_` |
| `ui-tars` | Tasks where you want the UI-TARS model | `tsk_v2_` |

**Response** (`TaskRunResponse`)

```json
{
    "run_id": "tsk_v2_abc123",
    "run_type": "task_v2",
    "status": "running",
    "output": null,
    "failure_reason": null,
    "created_at": "2025-01-15T10:00:00Z",
    "modified_at": "2025-01-15T10:00:01Z",
    "queued_at": "2025-01-15T10:00:00Z",
    "started_at": null,
    "finished_at": null,
    "app_url": "https://app.skyvern.com/runs/tsk_v2_abc123",
    "browser_session_id": null,
    "recording_url": null,
    "downloaded_files": null,
    "step_count": null
}
```

---

### Run a workflow

`POST /v1/run/workflows`

Runs a previously created workflow. The `workflow_id` comes from the `workflow_permanent_id` field on a `Workflow` object, and starts with `wpid_`.

```python
from skyvern import Skyvern

skyvern = Skyvern(api_key="YOUR_API_KEY")
run = await skyvern.run_workflow(
    workflow_id="wpid_123",
    parameters={"login_url": "https://example.com/login", "username": "user@example.com"},
)
print(run.run_id)   # wr_abc123
```

```typescript
import { SkyvernClient } from "@skyvern/client";

const skyvern = new SkyvernClient({ apiKey: "YOUR_API_KEY" });
const run = await skyvern.runWorkflow({
    body: {
        workflow_id: "wpid_123",
        parameters: { login_url: "https://example.com/login", username: "user@example.com" }
    }
});
```

**Request body**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `workflow_id` | `string` | required | The permanent ID of the workflow to run. Starts with `wpid_`. |
| `parameters` | `object \| null` | `null` | Key-value pairs matching the workflow's parameter definitions. |
| `title` | `string \| null` | `null` | Human-readable label for this workflow run. |
| `webhook_url` | `string \| null` | `null` | URL to POST a completion notification to. |
| `proxy_location` | `string \| object \| null` | `"RESIDENTIAL"` | Geographic proxy. See [proxy locations](#proxy-locations). |
| `totp_identifier` | `string \| null` | `null` | Identifier for incoming TOTP push codes. |
| `totp_url` | `string \| null` | `null` | URL Skyvern polls for TOTP codes. |
| `browser_session_id` | `string \| null` | `null` | Resume from an existing persistent browser session. |
| `browser_profile_id` | `string \| null` | `null` | Browser profile to load (cookies, localStorage, etc.). |
| `extra_http_headers` | `object \| null` | `null` | Additional headers injected into all browser requests. |
| `browser_address` | `string \| null` | `null` | CDP address of a custom browser. |
| `ai_fallback` | `boolean \| null` | `null` | If `true`, falls back to AI agent if code execution fails. |

**Response** (`WorkflowRunResponse`)

```json
{
    "run_id": "wr_abc123",
    "run_type": "workflow_run",
    "status": "running",
    "output": null,
    "failure_reason": null,
    "created_at": "2025-01-15T10:00:00Z",
    "modified_at": "2025-01-15T10:00:01Z",
    "app_url": "https://app.skyvern.com/runs/wr_abc123",
    "browser_session_id": null,
    "browser_profile_id": null
}
```

---

### Get a run

`GET /v1/runs/{run_id}`

Polls the current state of any run (task or workflow). The `run_id` is whatever was returned by `run_task` or `run_workflow`.

```python
from skyvern import Skyvern

skyvern = Skyvern(api_key="YOUR_API_KEY")
run = await skyvern.get_run(run_id="tsk_v2_abc123")
print(run.status)   # completed
print(run.output)   # {"posts": [...]}
```

```typescript
const run = await skyvern.getRun("tsk_v2_abc123");
```

**Path parameters**

| Parameter | Description |
|---|---|
| `run_id` | The run ID. Accepts `tsk_`, `tsk_v2_`, and `wr_` prefixed IDs. |

**Run statuses**

A run moves through these states:

- `created` → `queued` → `running` → `completed`
- Or terminates in: `failed`, `timed_out`, `terminated`, `canceled`

Once a run reaches `completed`, `failed`, `timed_out`, `terminated`, or `canceled` it will not change again.

**Response** (`RunResponse`, either `TaskRunResponse` or `WorkflowRunResponse` based on `run_type`)

```json
{
    "run_id": "tsk_v2_abc123",
    "run_type": "task_v2",
    "status": "completed",
    "output": {
        "posts": ["Post A", "Post B", "Post C"]
    },
    "failure_reason": null,
    "recording_url": "https://...",
    "screenshot_urls": ["https://...", "https://..."],
    "downloaded_files": [],
    "step_count": 4,
    "created_at": "2025-01-15T10:00:00Z",
    "modified_at": "2025-01-15T10:00:45Z",
    "started_at": "2025-01-15T10:00:02Z",
    "finished_at": "2025-01-15T10:00:45Z",
    "app_url": "https://app.skyvern.com/runs/tsk_v2_abc123"
}
```

`output` will be `null` while the run is still in progress. The shape of `output` matches the `data_extraction_schema` you provided (or is an unstructured string/object if no schema was given).

---

### Cancel a run

`POST /v1/runs/{run_id}/cancel`

Stops a run that is in `created`, `queued`, or `running` status. Has no effect on already-finished runs.

```python
await skyvern.cancel_run(run_id="tsk_v2_abc123")
```

```typescript
await skyvern.cancelRun("tsk_v2_abc123");
```

Returns `204 No Content` on success.

---

### Get run artifacts

`GET /v1/runs/{run_id}/artifacts`

Returns the raw artifacts collected during a run: screenshots, LLM logs, HTML snapshots, recordings, and more.

```python
artifacts = await skyvern.get_run_artifacts(run_id="tsk_v2_abc123")
for a in artifacts:
    print(a.artifact_type, a.signed_url)
```

**Query parameters**

| Parameter | Type | Description |
|---|---|---|
| `artifact_type` | `string[]` | Filter to specific types. Repeatable. |

**Common artifact types**

| Value | Description |
|---|---|
| `screenshot_final` | Final screenshot after the run completes |
| `screenshot_llm` | Screenshot sent to the LLM at each step |
| `screenshot_action` | Screenshot taken after each action |
| `recording` | Full browser session recording (WebM video) |
| `llm_prompt` | The prompt sent to the LLM |
| `llm_response` | The raw LLM response |
| `browser_console_log` | Browser console output |
| `skyvern_log` | Skyvern agent logs |

**Response** (`Artifact[]`)

```json
[
    {
        "artifact_id": "art_abc123",
        "artifact_type": "screenshot_final",
        "signed_url": "https://...",
        "task_id": "tsk_abc123",
        "run_id": "tsk_v2_abc123",
        "created_at": "2025-01-15T10:00:45Z",
        "modified_at": "2025-01-15T10:00:45Z",
        "organization_id": "org_abc123"
    }
]
```

`signed_url` is a time-limited URL you can open in a browser or download directly.

---

### Get run timeline

`GET /v1/runs/{run_id}/timeline`

Returns a step-by-step breakdown of what happened during a `task_v2` or `workflow_run`. Each entry in the timeline represents a block execution, showing what the agent did and whether it succeeded. Only available for `task_v2` (IDs starting with `tsk_v2_`) and workflow runs (`wr_`). Returns `400` for v1 task runs.

```python
timeline = await skyvern.get_run_timeline(run_id="wr_abc123")
for entry in timeline:
    print(entry)
```

---

### Retry run webhook

`POST /v1/runs/{run_id}/retry_webhook`

Re-sends the completion webhook for a finished run. Useful if your webhook endpoint was down or returned an error when the run completed.

```python
await skyvern.retry_run_webhook(run_id="tsk_v2_abc123")
```

**Request body** (optional)

| Parameter | Type | Description |
|---|---|---|
| `webhook_url` | `string \| null` | Override the destination URL. If omitted, uses the URL from the original run. |

---

## Workflows

Workflows are reusable automation definitions made of blocks. You create them once, then run them repeatedly with different parameters — making them ideal for scheduled jobs, batch processing, or any automation you want to version and manage separately from the code that invokes it.

### Create a workflow

`POST /v1/workflows`

Accepts a workflow definition in either YAML or JSON format. The `workflow_permanent_id` in the response is the stable identifier you use to run the workflow.

```python
from skyvern import Skyvern

skyvern = Skyvern(api_key="YOUR_API_KEY")
workflow = await skyvern.create_workflow(
    title="Fill Contact Form",
    description="Navigate to a URL and submit a contact form",
    workflow_definition={
        "parameters": [
            {
                "key": "website_url",
                "parameter_type": "workflow",
                "workflow_parameter_type": "string",
                "default_value": None,
            }
        ],
        "blocks": [
            {
                "label": "fill_form",
                "block_type": "navigation",
                "url": "{{website_url}}",
                "navigation_goal": "Find and submit the contact form.",
            }
        ],
    },
)
print(workflow.workflow_permanent_id)   # wpid_abc123
```

You can also pass raw YAML via the `yaml_definition` field in the JSON body, or send `Content-Type: text/plain` with the YAML body directly.

---

### Update a workflow

`POST /v1/workflows/{workflow_id}`

Replaces the definition of an existing workflow. Creates a new version — previous versions are kept and accessible via the versions endpoint.

| Path parameter | Description |
|---|---|
| `workflow_id` | The `workflow_permanent_id` of the workflow to update. Starts with `wpid_`. |

The request body has the same shape as [Create a workflow](#create-a-workflow).

---

### Delete a workflow

`POST /v1/workflows/{workflow_id}/delete`

Soft-deletes a workflow. Existing runs are not affected.

```python
await skyvern.delete_workflow(workflow_id="wpid_123")
```

---

### Get a workflow

`GET /v1/workflows/{workflow_permanent_id}`

Returns the latest version of a workflow.

```python
workflow = await skyvern.get_workflow(workflow_permanent_id="wpid_123")
```

**Query parameters**

| Parameter | Type | Description |
|---|---|---|
| `version` | `integer` | Return a specific version instead of the latest. |

---

### List workflows

`GET /v1/workflows`

Returns all workflows for your organization.

```python
workflows = await skyvern.get_workflows()
```

**Query parameters**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `page` | `integer` | `1` | Page number. |
| `page_size` | `integer` | `10` | Items per page. |
| `search_key` | `string` | — | Case-insensitive substring search across workflow title, folder name, and parameter metadata. |
| `folder_id` | `string` | — | Filter by folder. |
| `status` | `string[]` | `["published", "draft"]` | Filter by status. Options: `published`, `draft`. |

---

### List workflow runs

`GET /v1/workflows/runs`

Returns all workflow runs across all workflows for your organization.

**Query parameters**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `page` | `integer` | `1` | Page number. |
| `page_size` | `integer` | `10` | Items per page. |
| `status` | `string[]` | — | Filter by one or more statuses. |
| `search_key` | `string` | — | Search across run ID, parameter keys/values, and extra headers. |
| `error_code` | `string` | — | Exact match on the `error_code` in a run's error array (e.g. `LOGIN_FAILED`). |

---

## Credentials

Credentials let Skyvern log into sites on your behalf without you ever passing raw secrets through the browser automation. When you reference a credential in a task or workflow, Skyvern fetches it from the vault at runtime. Credential responses never include the raw secret — only non-sensitive metadata like username, card brand, or last four digits.

### Create a credential

`POST /v1/credentials`

Skyvern supports three credential types: `password` for site logins, `credit_card` for payment forms, and `secret` for arbitrary tokens or API keys.

```python
from skyvern import Skyvern

skyvern = Skyvern(api_key="YOUR_API_KEY")

# Password credential
await skyvern.create_credential(
    name="My Site Login",
    credential_type="password",
    credential={"username": "user@example.com", "password": "s3cr3t"},
)

# Credit card credential
await skyvern.create_credential(
    name="My Visa",
    credential_type="credit_card",
    credential={
        "card_number": "4242424242424242",
        "card_cvv": "424",
        "card_exp_month": "12",
        "card_exp_year": "2028",
        "card_brand": "visa",
        "card_holder_name": "Jane Doe",
    },
)

# Secret credential (API keys, tokens, etc.)
await skyvern.create_credential(
    name="API Token",
    credential_type="secret",
    credential={"secret_value": "sk-abc123", "secret_label": "Bearer Token"},
)
```

**Request body**

| Parameter | Type | Description |
|---|---|---|
| `name` | `string` | Human-readable name for the credential. |
| `credential_type` | `string` | One of: `password`, `credit_card`, `secret`. |
| `credential` | `object` | The credential payload. Shape depends on `credential_type` (see below). |

**`password` credential fields**

| Field | Type | Required | Description |
|---|---|---|---|
| `username` | `string` | yes | Login username or email. |
| `password` | `string` | yes | Password. |
| `totp` | `string \| null` | no | Authenticator app secret (base-32 TOTP seed). |
| `totp_type` | `string` | no | One of: `authenticator`, `email`, `text`, `none`. Default: `none`. |
| `totp_identifier` | `string \| null` | no | Identifier used to match incoming TOTP push codes. |

**`credit_card` credential fields**

| Field | Type | Required | Description |
|---|---|---|---|
| `card_number` | `string` | yes | Full card number. |
| `card_cvv` | `string` | yes | CVV. |
| `card_exp_month` | `string` | yes | Expiration month (e.g. `"12"`). |
| `card_exp_year` | `string` | yes | Expiration year (e.g. `"2028"`). |
| `card_brand` | `string` | yes | Card brand (e.g. `"visa"`, `"mastercard"`). |
| `card_holder_name` | `string` | yes | Name on the card. |

**`secret` credential fields**

| Field | Type | Required | Description |
|---|---|---|---|
| `secret_value` | `string` | yes | The secret string. |
| `secret_label` | `string \| null` | no | Optional description of what the secret is. |

**Response** (`CredentialResponse`)

```json
{
    "credential_id": "cred_abc123",
    "name": "My Site Login",
    "credential_type": "password",
    "credential": {
        "username": "user@example.com",
        "totp_type": "none",
        "totp_identifier": null
    },
    "browser_profile_id": null,
    "tested_url": null
}
```

---

### Get a credential

`GET /v1/credentials/{credential_id}`

```python
credential = await skyvern.get_credential(credential_id="cred_abc123")
```

---

### List credentials

`GET /v1/credentials`

```python
credentials = await skyvern.get_credentials()
```

---

### Delete a credential

`POST /v1/credentials/{credential_id}/delete`

```python
await skyvern.delete_credential(credential_id="cred_abc123")
```

---

### Send TOTP code

`POST /v1/credentials/totp`

Forwards a 2FA/MFA code to Skyvern mid-run. Use this when your 2FA arrives via email or SMS and you need to push the code to a running task or workflow.

The flow: start a task with `totp_identifier` set to your email or phone number. When the 2FA message arrives, forward its full text here — Skyvern parses the numeric code automatically.

```python
await skyvern.send_totp_code(
    totp_identifier="user@example.com",
    content="Your verification code is 847392. It expires in 10 minutes.",
)
```

**Request body**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `totp_identifier` | `string` | yes | The identifier you set on the task/workflow (email, phone, or custom string). |
| `content` | `string` | yes | The full message containing the code. Skyvern extracts the numeric code automatically. |
| `task_id` | `string \| null` | no | Scopes the code to a specific task run. |
| `workflow_run_id` | `string \| null` | no | Scopes the code to a specific workflow run. |
| `source` | `string \| null` | no | Where the code came from: `"email"`, `"sms"`, etc. |
| `expired_at` | `datetime \| null` | no | When this code expires. |

---

## Browser Sessions

A browser session keeps a browser instance alive across multiple runs. Instead of starting a fresh browser every time, runs using the same session pick up from wherever the previous run left off — useful for multi-step flows that require staying logged in.

Session IDs start with `pbs_`.

### Create a session

`POST /v1/browser_sessions`

```python
from skyvern import Skyvern

skyvern = Skyvern(api_key="YOUR_API_KEY")
session = await skyvern.create_browser_session(timeout=60)
print(session.browser_session_id)   # pbs_abc123
print(session.browser_address)      # https://... (CDP address)
```

**Request body**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `timeout` | `integer` | `60` | Session timeout in minutes. Min: 5, max: 1440. The session closes automatically after this long without activity. |
| `proxy_location` | `string \| null` | `null` | Geographic proxy for the session. See [proxy locations](#proxy-locations). |
| `browser_profile_id` | `string \| null` | `null` | Load a saved browser profile (cookies, localStorage) into the session. |
| `browser_type` | `string \| null` | `null` | Browser type. |
| `extensions` | `array \| null` | `null` | Browser extensions to install. |

**Response** (`BrowserSessionResponse`)

```json
{
    "browser_session_id": "pbs_abc123",
    "organization_id": "org_abc123",
    "status": "created",
    "timeout": 60,
    "browser_address": "https://3.12.10.11/browser/abc123",
    "app_url": "https://app.skyvern.com/browser-session/pbs_abc123",
    "browser_profile_id": null,
    "vnc_streaming_supported": false,
    "created_at": "2025-01-15T10:00:00Z",
    "modified_at": "2025-01-15T10:00:00Z"
}
```

Once you have a `browser_session_id`, pass it to `run_task` or `run_workflow` via the `browser_session_id` parameter. Each subsequent run using that session continues from the browser's current state.

---

### Get a session

`GET /v1/browser_sessions/{browser_session_id}`

```python
session = await skyvern.get_browser_session(browser_session_id="pbs_abc123")
print(session.status)   # running | completed | timeout
```

---

### List sessions

`GET /v1/browser_sessions`

```python
sessions = await skyvern.get_browser_sessions()
```

**Query parameters**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `page` | `integer` | `1` | Page number. |
| `page_size` | `integer` | `10` | Items per page. |

---

### Close a session

`POST /v1/browser_sessions/{browser_session_id}/close`

Terminates the session. Closed sessions cannot be reused.

```python
await skyvern.close_browser_session(browser_session_id="pbs_abc123")
```

---

## Proxy Locations

The `proxy_location` parameter is available on tasks, workflow runs, and browser sessions. All residential proxies are only available on Skyvern Cloud.

| Value | Description |
|---|---|
| `RESIDENTIAL` | Default. Random US residential IP. |
| `RESIDENTIAL_ES` | Spain |
| `RESIDENTIAL_IE` | Ireland |
| `RESIDENTIAL_GB` | United Kingdom |
| `RESIDENTIAL_IN` | India |
| `RESIDENTIAL_JP` | Japan |
| `RESIDENTIAL_FR` | France |
| `RESIDENTIAL_DE` | Germany |
| `RESIDENTIAL_AU` | Australia |
| `RESIDENTIAL_BR` | Brazil |
| `RESIDENTIAL_CA` | Canada |
| `RESIDENTIAL_MX` | Mexico |
| `RESIDENTIAL_IT` | Italy |
| `RESIDENTIAL_NL` | Netherlands |
| `RESIDENTIAL_KR` | South Korea |
| `RESIDENTIAL_ISP` | ISP proxy (US) |
| `NONE` | No proxy — browser uses the server's network directly. |

For city or state-level targeting, pass an object instead of a string:

```json
{
    "proxy_location": {
        "country": "US",
        "subdivision": "CA",
        "city": "San Francisco"
    }
}
```

`country` is required (ISO 3166-1 alpha-2). `subdivision` and `city` are optional refinements.

---

## Errors

The API returns standard HTTP status codes. Error bodies follow this shape:

```json
{
    "detail": "Task run not found tsk_v2_abc123"
}
```

| Status | Meaning |
|---|---|
| `400` | Bad request — invalid parameters or request body. |
| `403` | Forbidden — missing or invalid `x-api-key`. |
| `404` | Not found — the resource doesn't exist or belongs to a different organization. |
| `422` | Unprocessable entity — validation failure (e.g. malformed YAML workflow definition). |
| `500` | Internal server error — something went wrong on Skyvern's side. |

Runs that fail due to agent behavior (wrong credentials, site unreachable, etc.) do not return HTTP errors. They return `200` with `status: "failed"` and a `failure_reason` string explaining what happened.
