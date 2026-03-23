# Quickstart

Get Skyvern running and automate your first browser task in under five minutes.

## Prerequisites

- Python 3.11, 3.12, or 3.13
- A Skyvern account with an API key ([app.skyvern.com/settings](https://app.skyvern.com/settings))

If you want to run the browser locally on your machine (so you can watch it work), you'll also need Node.js and npm for the browser install step.

---

## Choose your setup path

**Use Skyvern Cloud (Option A)** if you want the fastest start: your code runs locally, the browser runs on Skyvern's infrastructure, and there's no local server to configure.

**Run locally (Option B)** if you want to watch the browser in real time, automate sites on a private network, or debug your prompts interactively. This requires a two-minute setup to configure an LLM provider and install Chromium.

---

## Option A: Skyvern Cloud (no local setup)

### 1. Install the SDK

```bash
pip install skyvern
```

### 2. Run your first task

```python
import asyncio
from skyvern import Skyvern

skyvern = Skyvern(api_key="YOUR_API_KEY")

task = asyncio.run(
    skyvern.run_task(
        prompt="Go to news.ycombinator.com and return the titles of the top 3 posts.",
        url="https://news.ycombinator.com",
        wait_for_completion=True,  # bool, default False — blocks until the task reaches a final status
    )
)

print(task.status)   # "completed"
print(task.output)   # {"answer": "1. ... 2. ... 3. ..."}
```

Replace `YOUR_API_KEY` with the key from your settings page. When you run this, Skyvern opens a browser session on its cloud servers, navigates to Hacker News, reads the page, and returns the result. You don't need to think about browsers, proxies, or Playwright.

The returned `TaskRunResponse` object has these fields:

| Field | Type | Description |
|---|---|---|
| `run_id` | `str` | Unique identifier (`tsk_…` for tasks, `wr_…` for workflows) |
| `status` | `str` | `"created"` → `"queued"` → `"running"` → `"completed"` / `"failed"` / `"terminated"` / `"timed_out"` / `"canceled"` |
| `output` | `dict \| None` | Extracted data; `None` until the task completes |
| `failure_reason` | `str \| None` | Human-readable explanation when `status` is `"failed"` or `"terminated"` |
| `recording_url` | `str \| None` | URL to a video recording of the browser session |
| `app_url` | `str \| None` | Link to the run in the Skyvern dashboard |

### 3. Verify it worked

`task.status` should be `"completed"` and `task.output` will contain the extracted text. Open `task.app_url` in your browser to watch a recording of exactly what the agent did.

---

## Option B: Run locally (browser opens on your machine)

### 1. Install the SDK and run setup

```bash
pip install skyvern
skyvern quickstart
```

The `quickstart` command walks you through selecting an LLM provider, sets your API keys, and writes a `.env` file. It also installs Chromium via Playwright. Follow the prompts — the whole thing takes about two minutes.

### 2. Run your first task

```python
import asyncio
from skyvern import Skyvern

async def main():
    skyvern = Skyvern.local()  # reads your .env file; raises if it doesn't exist

    browser = await skyvern.launch_local_browser()
    page = await browser.get_working_page()

    await page.goto("https://news.ycombinator.com")

    result = await page.agent.run_task(
        "Return the titles of the top 3 posts on this page."
    )

    print(result.status)   # "completed"
    print(result.output)   # {"answer": "1. ... 2. ... 3. ..."}

    await browser.close()

asyncio.run(main())
```

A Chromium window will open, navigate to Hacker News, and close when the task finishes. `Skyvern.local()` reads the `.env` file created by `skyvern quickstart`. If that file doesn't exist, it raises an error pointing you back to setup.

### 3. Verify it worked

The browser window closes on its own when the task completes. `result.status` should be `"completed"`.

---

## Extract structured data

Both options support `data_extraction_schema` for getting typed JSON back instead of free-form text. This is the parameter to reach for any time you need your automation to feed data into another system.

```python
task = asyncio.run(
    skyvern.run_task(
        prompt="Extract the top 3 posts from Hacker News.",
        url="https://news.ycombinator.com",
        data_extraction_schema={
            "type": "array",
            "items": {
                "type": "object",
                "properties": {
                    "title": {"type": "string"},
                    "points": {"type": "integer"},
                    "url": {"type": "string"},
                },
                "required": ["title", "points"],
            },
        },
        wait_for_completion=True,
    )
)

print(task.output)
# [{"title": "...", "points": 412, "url": "..."}, ...]
```

The schema follows [JSON Schema](https://json-schema.org/) format. Skyvern uses it to shape the LLM's output, so you can treat `task.output` as typed data rather than parsing a string.

---

## Mix AI actions with Playwright

When using a local browser (Option B), you get a full Playwright `page` object with extra AI-powered methods layered on top. You can mix standard Playwright calls with Skyvern's AI methods in the same script:

```python
# Standard Playwright: navigate and fill fields by selector
await page.goto("https://example.com/login")
await page.fill("#email", "you@example.com")

# AI-powered: handle anything you don't want to hard-code
await page.act("Click the Sign In button")          # returns None
logged_in = await page.validate("Confirm the user is now logged in")
print(logged_in)  # True or False

# Extract structured data from whatever is on screen
data = await page.extract(
    "Get the account balance shown on this page",
    schema={"type": "object", "properties": {"balance": {"type": "number"}}},
)
print(data)  # {"balance": 1234.56}
```

`page.act(prompt)` performs an action described in plain English and returns `None`. `page.validate(prompt)` returns `True` or `False`. `page.extract(prompt, schema)` returns a dict shaped by the schema you provide. These three methods cover most situations where you would otherwise need brittle selectors or custom scraping logic.

---

## TypeScript

If you're working in TypeScript, install `@skyvern/client` instead:

```bash
npm install @skyvern/client
```

```typescript
import { SkyvernClient } from "@skyvern/client";

const client = new SkyvernClient({ apiKey: "YOUR_API_KEY" });

const task = await client.runTask({
    body: {
        prompt: "Return the top 3 posts from Hacker News.",
        url: "https://news.ycombinator.com",
    },
});

console.log(task.runId);   // "tsk_..."
console.log(task.status);  // "running" — poll until this is a final status
```

The TypeScript client is a thin API wrapper and does not have `wait_for_completion` built in. Poll `client.getRun(runId)` until `status` reaches a final state:

```typescript
const FINAL_STATUSES = new Set(["completed", "failed", "terminated", "timed_out", "canceled"]);

async function waitForRun(client: SkyvernClient, runId: string, intervalMs = 2000) {
    while (true) {
        const run = await client.getRun(runId);
        if (FINAL_STATUSES.has(run.status)) return run;
        await new Promise(r => setTimeout(r, intervalMs));
    }
}

const result = await waitForRun(client, task.runId);
console.log(result.status);  // "completed"
console.log(result.output);  // {"answer": "..."}
```

---

## Next steps

- **[Run Tasks](../running-tasks/run-tasks.md)**: all `run_task` parameters, polling patterns, and webhooks
- **[Credentials](../credentials/overview.md)**: store and reuse passwords and 2FA secrets securely so the agent can log in to real sites
- **[Workflows](../workflows/manage-workflows.md)**: chain multiple tasks into reusable, parameterized automation
- **[Browser Sessions](../browser-sessions/introduction.md)**: keep a browser alive across multiple task runs to maintain login state
