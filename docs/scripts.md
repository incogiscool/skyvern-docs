# Scripts

Scripts are Python files that run inside Skyvern's execution environment. They give you direct control over browser automation using a `SkyvernPage` object — a Playwright `Page` with AI-powered methods added on top. Use scripts when you want code-level control over the automation rather than natural language prompts, when you have performance requirements, or when you're building something too complex for a single workflow block.

## How scripts differ from workflows

Workflows describe automation in YAML: blocks execute sequentially, each block translates into a browser task or LLM call at runtime. Scripts are executable Python code: you call functions, handle logic yourself, and interact directly with the browser page object.

The two approaches are complementary. A workflow can contain a script block that calls hand-written code. A script can call Skyvern's action functions (`run_task`, `extract`, `login`) which use the same AI-powered browser automation as workflow blocks.

A key advantage of scripts is caching. When Skyvern generates a script from a workflow, it can replay exact browser interactions (recorded from a previous successful run) instead of asking the LLM again. This makes repeated executions much faster.

## The SkyvernPage object

Scripts receive a `SkyvernPage` object that extends Playwright's `Page` class. You have access to all standard Playwright methods plus these AI-powered additions:

**`page.act(prompt)`** — Perform an action using natural language. The agent takes a screenshot and executes whatever the prompt describes.

```python
await page.act("Click the Submit button")
await page.act("Fill in the email field with the user's address")
```

**`page.extract(prompt, schema=None)`** — Extract structured data from the current page.

```python
result = await page.extract("Get the product name and price")

# With a JSON Schema for structured output:
result = await page.extract(
    prompt="Extract all order rows",
    schema={
        "type": "array",
        "items": {
            "type": "object",
            "properties": {
                "order_id": {"type": "string"},
                "amount": {"type": "number"}
            }
        }
    }
)
```

**`page.validate(prompt)`** — Check page state and return a boolean.

```python
is_logged_in = await page.validate("Check if the user is logged in")
if not is_logged_in:
    raise Exception("Login failed")
```

**`page.prompt(prompt, schema=None)`** — Send an arbitrary prompt to the LLM and return its response.

In addition to these AI methods, all standard Playwright methods work normally:

```python
await page.goto("https://example.com")
await page.click("#submit-button")          # selector-based
await page.click(prompt="Submit button")    # AI-based
await page.fill("#email", "user@example.com")  # selector-based
await page.fill(prompt="Email field", value="user@example.com")  # AI-based
```

## Action functions

Scripts can also call high-level action functions imported from `skyvern`. These mirror the workflow block types and handle all the orchestration (browser lifecycle, retry logic, artifact creation) for you.

```python
import skyvern

# Navigate and complete a task
result = await skyvern.run_task(
    prompt="Log in with the provided credentials",
    url="https://example.com/login",
    label="login_step",
)

# Extract data
data = await skyvern.extract(
    prompt="Extract all invoice rows",
    schema={"type": "array"},
    url="https://example.com/invoices",
    label="extract_invoices",
)

# Log in using stored credentials
await skyvern.login(
    prompt="Log in with credentials",
    url="https://example.com/login",
    label="login",
)

# Download a file
await skyvern.download(
    prompt="Click the Export button and download the CSV",
    url="https://example.com/reports",
    label="download_report",
)

# Make an HTTP request
response = await skyvern.http_request(
    url="https://api.example.com/data",
    method="GET",
    headers={"Authorization": "Bearer token"},
    label="api_call",
)

# Wait for a fixed duration
await skyvern.wait(seconds=5, label="pause")
```

Available action functions: `run_task`, `action`, `login`, `extract`, `download`, `validate`, `wait`, `goto`, `run_script`, `run_code`, `send_email`, `upload_file`, `parse_pdf`, `parse_file`, `http_request`, `render_template`, `render_list`.

## The @cached decorator

When Skyvern generates a script from a workflow, each block is wrapped with `@cached`. A cached function holds the exact browser interactions recorded during a previous successful run. When the script runs again, Skyvern replays those interactions instead of calling the LLM — skipping the screenshot/LLM/action loop entirely.

```python
import skyvern
from skyvern import cached, SkyvernPage, RunContext

@cached("login_step")
async def login_step(page: SkyvernPage, context: RunContext) -> None:
    await page.goto("https://example.com/login")
    await page.fill("#email", context.parameters["username"])
    await page.fill("#password", context.parameters["password"])
    await page.click("#submit")
```

The `cache_key` argument to `@cached` matches the block `label` in the workflow. If the cached function raises an exception (the site layout changed, a selector is stale), Skyvern automatically falls back to the AI-powered run for that block and can regenerate the cached version.

## Setting up a script

A script is a Python file with a main async function decorated with `@workflow`. The `@workflow` decorator registers the script with the execution environment.

```python
import skyvern
from skyvern import workflow, SkyvernPage, RunContext

@workflow(title="Extract Hacker News Posts")
async def main(page: SkyvernPage, context: RunContext) -> None:
    result = await skyvern.extract(
        prompt="Extract the top 5 posts with titles and scores",
        url="https://news.ycombinator.com",
        schema={
            "type": "array",
            "items": {
                "type": "object",
                "properties": {
                    "title": {"type": "string"},
                    "score": {"type": "integer"}
                }
            }
        },
        label="scrape_hn",
    )
    print(result)
```

Run it locally with `skyvern run server`, then execute the script via the API or SDK.

## Creating and deploying a script via the API

Scripts are stored as versioned files. Create one by posting base64-encoded file content:

```bash
curl -X POST https://api.skyvern.com/api/v1/scripts \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "files": [
      {
        "path": "main.py",
        "content": "<base64-encoded Python>",
        "encoding": "base64",
        "mime_type": "text/x-python"
      }
    ]
  }'
```

Run it:

```bash
curl -X POST https://api.skyvern.com/api/v1/scripts/{script_id}/run \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'
```

## When to use scripts vs. workflows

Use **workflows** when:
- The automation is straightforward and maps cleanly to sequential steps
- Non-technical team members need to read or edit the automation
- You want the UI's visual workflow builder

Use **scripts** when:
- You need custom logic (conditionals, data transformation, error handling) beyond what workflow blocks provide
- You're optimizing for speed and want fine-grained control over caching
- You're integrating Skyvern into a larger Python codebase and want first-class Python APIs
- You need to mix Playwright selector-based interactions with AI-powered ones in the same function
