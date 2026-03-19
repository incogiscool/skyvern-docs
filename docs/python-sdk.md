# Python SDK

The Skyvern Python SDK (`pip install skyvern`) gives you programmatic access to browser automation. It wraps the HTTP API but adds three important things: browser management (launching and connecting to browsers), a `SkyvernPage` object that extends Playwright with AI-powered methods, and convenience helpers like automatic polling until a task completes.

## Installation

```bash
pip install skyvern
```

Python 3.11 or 3.12 is required.

## Creating a client

There are two ways to create a `Skyvern` client, depending on whether you're running locally or using Skyvern Cloud.

**Local mode** (requires `skyvern quickstart` to have been run):

```python
from skyvern import Skyvern

skyvern = Skyvern.local()
```

`Skyvern.local()` reads your API key and settings from the `.env` file created by `skyvern quickstart`. It runs an embedded server in the same process, so no separate server is needed.

**Cloud or self-hosted mode:**

```python
from skyvern import Skyvern

skyvern = Skyvern(api_key="your-api-key")
```

Point at a self-hosted instance by passing `base_url`:

```python
skyvern = Skyvern(
    api_key="your-api-key",
    base_url="http://localhost:8000",
)
```

## Running tasks

`run_task` submits a task and optionally waits for it to complete.

```python
import asyncio
from skyvern import Skyvern

async def main():
    skyvern = Skyvern(api_key="your-api-key")

    # Submit and wait for completion (wait_for_completion=True)
    task = await skyvern.run_task(
        prompt="Find the top 3 posts on Hacker News and return their titles",
        wait_for_completion=True,
    )
    print(task.status)   # "completed"
    print(task.output)   # the extracted data
```

For structured output, pass a JSON Schema:

```python
task = await skyvern.run_task(
    prompt="Get the top 5 posts on Hacker News",
    data_extraction_schema={
        "type": "array",
        "items": {
            "type": "object",
            "properties": {
                "title": {"type": "string"},
                "score": {"type": "integer"},
                "url": {"type": "string"}
            }
        }
    },
    wait_for_completion=True,
    timeout=120.0,
)
```

**`run_task` parameters:**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `prompt` | str | required | Natural language goal |
| `url` | str | `None` | Starting URL |
| `engine` | RunEngine | `skyvern_v2` | Execution engine |
| `data_extraction_schema` | dict/str | `None` | JSON Schema for structured output |
| `proxy_location` | ProxyLocation | `None` | Proxy region |
| `max_steps` | int | `None` | Step limit |
| `webhook_url` | str | `None` | Webhook for completion notification |
| `totp_identifier` | str | `None` | TOTP email/phone identifier |
| `totp_url` | str | `None` | URL to poll for TOTP codes |
| `browser_session_id` | str | `None` | Reuse a persistent browser session |
| `browser_address` | str | `None` | CDP URL for external browser |
| `wait_for_completion` | bool | `False` | Block until task finishes |
| `timeout` | float | `1800.0` | Seconds to wait if `wait_for_completion=True` |

The response is a `TaskRunResponse` with `run_id`, `status`, and `output`.

## Running workflows

```python
workflow_run = await skyvern.run_workflow(
    workflow_id="wpid_abc123",
    parameters={
        "target_url": "https://example.com",
        "report_email": "team@company.com",
    },
    wait_for_completion=True,
)
print(workflow_run.status)
print(workflow_run.output)
```

## Polling for results

When you don't use `wait_for_completion=True`, you can poll manually:

```python
import asyncio

task = await skyvern.run_task(prompt="...")

while True:
    run = await skyvern.get_run(task.run_id)
    if run.status in ("completed", "failed", "terminated", "timed_out", "canceled"):
        break
    await asyncio.sleep(5)

print(run.output)
```

## Browser-based automation

For cases where you want to mix Playwright actions with AI-powered tasks in the same browser session, use the browser API.

### Launching a browser

**Local browser:**

```python
skyvern = Skyvern.local()
browser = await skyvern.launch_local_browser(headless=False)
page = await browser.get_working_page()
```

**Cloud browser** (Skyvern Cloud only):

```python
skyvern = Skyvern(api_key="your-api-key")
browser = await skyvern.launch_cloud_browser(proxy_location=ProxyLocation.RESIDENTIAL)
page = await browser.get_working_page()
```

`use_cloud_browser()` is like `launch_cloud_browser()` but reuses an existing session if one is available.

### AI page methods

Once you have a `page` (a `SkyvernBrowserPage` wrapping Playwright), you can mix standard Playwright calls with AI-powered actions:

```python
await page.goto("https://example.com")

# Standard Playwright
await page.click("#submit-button")
await page.fill("#email", "user@example.com")

# AI-powered (natural language)
await page.act("Click the login button")
await page.fill(prompt="Email field", value="user@example.com")

# Extract structured data
data = await page.extract(
    prompt="Get the product name and price",
    schema={"type": "object", "properties": {"name": {"type": "string"}, "price": {"type": "number"}}}
)

# Validate page state
is_logged_in = await page.validate("Is the user logged in?")

# Run a multi-step AI task in the current page context
task = await page.agent.run_task("Complete the checkout form and submit")
```

### page.agent methods

`page.agent` gives you higher-level workflow commands that operate in the context of the current browser session:

**`page.agent.run_task(prompt, ...)`** — Run an AI task on the current page. The agent sees the same browser state as the page.

**`page.agent.login(credential_type, ...)`** — Log in using stored credentials. Supports Skyvern credentials, Bitwarden, 1Password, and Azure Vault:

```python
from skyvern.schemas.run_blocks import CredentialType

# Skyvern credential
await page.agent.login(
    credential_type=CredentialType.skyvern,
    credential_id="cred_abc123",
)

# Bitwarden
await page.agent.login(
    credential_type=CredentialType.bitwarden,
    bitwarden_item_id="item_id",
    bitwarden_collection_id="collection_id",
)

# 1Password
await page.agent.login(
    credential_type=CredentialType.onepassword,
    onepassword_vault_id="vault_id",
    onepassword_item_id="item_id",
)
```

**`page.agent.run_workflow(workflow_id, parameters={})`** — Run a saved workflow in the current browser context.

**`page.agent.download_files(prompt)`** — Navigate and download files.

### Managing a session

```python
# Close the browser when done
await browser.close()
```

Persistent sessions survive `browser.close()` and can be reused:

```python
# Launch with a saved session
browser_session = await skyvern.create_browser_session()
browser = await skyvern._connect_to_cloud_browser_session(browser_session)
# ... do work ...
# Session remains active until explicitly terminated
```

## Canceling a run

```python
await skyvern.cancel_run(run_id="tsk_abc123")
```

## Credential management

```python
from skyvern.forge.sdk.schemas.credentials import CredentialType

# Create a password credential
credential = await skyvern.create_credential(
    name="My Login",
    credential_type="password",
    credential={"username": "user@example.com", "password": "secret"}
)
print(credential.credential_id)

# List credentials
credentials = await skyvern.list_credentials()

# Delete a credential
await skyvern.delete_credential(credential_id="cred_abc123")
```

## Full working example

```python
import asyncio
from skyvern import Skyvern
from skyvern.schemas.runs import ProxyLocation

async def main():
    skyvern = Skyvern(api_key="your-api-key")

    # Launch a cloud browser
    browser = await skyvern.launch_cloud_browser(
        proxy_location=ProxyLocation.RESIDENTIAL
    )
    page = await browser.get_working_page()

    # Navigate and log in with a stored credential
    await page.goto("https://shop.example.com/login")
    await page.agent.login(
        credential_type="skyvern",
        credential_id="cred_abc123",
    )

    # Mix Playwright and AI actions
    await page.click("#my-orders")
    orders = await page.extract(
        prompt="Get all orders from the last 30 days",
        schema={
            "type": "array",
            "items": {"type": "object", "properties": {
                "order_id": {"type": "string"},
                "total": {"type": "number"},
                "date": {"type": "string"}
            }}
        }
    )
    print(orders)

    await browser.close()

asyncio.run(main())
```

## SDK vs direct API calls

The Python SDK wraps the HTTP API but adds:
- Browser lifecycle management (`launch_local_browser`, `launch_cloud_browser`)
- A `SkyvernPage` object with `act()`, `extract()`, `validate()` — no API calls needed for these
- `wait_for_completion=True` for blocking until task/workflow finishes
- Automatic session handling when using `page.agent`

If you only need to submit tasks and retrieve results (no browser control), the raw HTTP API or the TypeScript SDK are equally good options.
