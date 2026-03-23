# Quickstart

Get Skyvern running and automate your first browser task in under five minutes.

## Prerequisites

- Python 3.11, 3.12, or 3.13
- A Skyvern account with an API key ([app.skyvern.com/settings](https://app.skyvern.com/settings))

If you want to run the browser locally on your machine (so you can watch it work), you'll also need Node.js and npm for the browser install step.

---

## Option A: Skyvern Cloud (no local setup)

The fastest path. Your code runs locally, the browser runs on Skyvern's infrastructure.

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
        wait_for_completion=True,
    )
)

print(task.status)   # "completed"
print(task.output)   # {"answer": "1. ... 2. ... 3. ..."}
```

Replace `YOUR_API_KEY` with the key from your settings page. When you run this, Skyvern opens a browser session on its cloud servers, navigates to Hacker News, reads the page, and returns the result. You don't need to think about browsers, proxies, or Playwright.

### 3. Verify it worked

`task.status` should be `"completed"` and `task.output` will contain the extracted text. You can also open `task.app_url` in your browser to watch a recording of exactly what the agent did.

---

## Option B: Run locally (browser opens on your machine)

Use this when you want to see the browser in real time, access sites on a local network, or debug your prompts interactively.

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
    skyvern = Skyvern.local()  # reads your .env file

    browser = await skyvern.launch_local_browser()
    page = await browser.get_working_page()

    await page.goto("https://news.ycombinator.com")

    result = await page.agent.run_task(
        "Return the titles of the top 3 posts on this page."
    )

    print(result.status)
    print(result.output)

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
await page.act("Click the Sign In button")

# Validate the result with AI
logged_in = await page.validate("Confirm the user is now logged in")
print(logged_in)  # True or False

# Extract structured data from whatever is on screen
data = await page.extract(
    "Get the account balance shown on this page",
    schema={"type": "object", "properties": {"balance": {"type": "number"}}},
)
print(data)  # {"balance": 1234.56}
```

`page.act()` performs an action described in plain English. `page.validate()` returns `True` or `False`. `page.extract()` returns structured data. These three methods cover most situations where you would otherwise need brittle selectors or custom scraping logic.

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

console.log(task.runId);   // tsk_...
console.log(task.status);  // "running" — poll getRun() until this is a final status
```

The TypeScript client is a thin API wrapper. Unlike the Python `Skyvern` class, it does not have `wait_for_completion` built in, so you poll `client.getRun(runId)` until `status` reaches a final state: `"completed"`, `"failed"`, `"terminated"`, `"timed_out"`, or `"canceled"`.

---

## Next steps

- **[Run Tasks](../running-tasks/run-tasks.md)** — all `run_task` parameters, polling patterns, and webhooks
- **[Credentials](../credentials/overview.md)** — store and reuse passwords and 2FA secrets securely so the agent can log in to real sites
- **[Workflows](../workflows/manage-workflows.md)** — chain multiple tasks into reusable, parameterized automation
- **[Browser Sessions](../browser-sessions/introduction.md)** — keep a browser alive across multiple task runs to maintain login state
