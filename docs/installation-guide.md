# Installation Guide

Skyvern is an AI-powered browser automation platform. This guide gets you from zero to running your first automation in a few minutes, whether you want to use Skyvern Cloud or run everything locally.

## Prerequisites

Before you install anything, decide which mode you want to use:

- **Skyvern Cloud** (simplest): Your Python/TypeScript code sends tasks to Skyvern's servers. No local infrastructure needed. You need a Skyvern account and API key.
- **Local mode**: Skyvern runs on your machine in-process. Your LLM API keys stay local. You need Python 3.11–3.13 and Node.js installed.

---

## Option 1: Skyvern Cloud

Create an account at [app.skyvern.com](https://app.skyvern.com), then grab your API key from the Settings page.

Install the Python SDK:

```bash
pip install skyvern
```

That's the entire setup. The SDK connects to `https://api.skyvern.com` by default. You authenticate by passing your key when you create a client:

```python
from skyvern import Skyvern

client = Skyvern(api_key="your-api-key")
```

The `Skyvern` class in `skyvern.library` is the main entry point. It inherits from `AsyncSkyvern` (the lower-level Fern-generated client), so it has the full API surface plus higher-level browser helpers like `launch_cloud_browser()`.

If you prefer TypeScript, install the npm package instead:

```bash
npm install @skyvern/client
```

Then initialize the client the same way:

```typescript
import { SkyvernClient } from "@skyvern/client";

const client = new SkyvernClient({ apiKey: "your-api-key" });
```

---

## Option 2: Local mode

Local mode runs an embedded Skyvern server in-process. There's no separate Docker daemon or external service to manage. What runs locally: the Skyvern agent, an SQLite database, and a Playwright-controlled Chromium browser. What you supply: an LLM API key (OpenAI, Anthropic, Gemini, or others).

### Step 1: Install Python dependencies

Python 3.11, 3.12, or 3.13 is required. Python 3.13 support is still in progress as of this writing, so 3.11 or 3.12 is the safe choice.

```bash
pip install skyvern
```

On **Windows** only, you also need [Rust](https://rustup.rs/) and Visual Studio with the "Desktop development with C++" workload installed before running `pip install skyvern`.

### Step 2: Run the quickstart

```bash
skyvern quickstart
```

This command walks you through the first-time setup interactively. It:

1. Prompts you to choose a setup method (in-process or Docker Compose).
2. Asks which LLM provider you want to use and where to find its API key.
3. Creates a `.env` file in your current directory with all the configuration.
4. Installs Playwright browsers (`playwright install chromium`).
5. Sets up the local SQLite database.

After it completes, the `.env` file contains a locally generated `SKYVERN_API_KEY` value alongside your LLM configuration. That key is what `Skyvern.local()` reads automatically.

### Step 3: Verify the setup

Create a quick test script in the same directory as the `.env` file:

```python
import asyncio
from skyvern import Skyvern

async def main():
    skyvern = Skyvern.local()
    browser = await skyvern.launch_local_browser(headless=False)
    page = await browser.get_working_page()

    await page.goto("https://example.com")
    title = await page.title()
    print(f"Page title: {title}")

    await browser.close()

asyncio.run(main())
```

Run it with `python test.py`. A Chromium window should open, navigate to example.com, and print the page title. If that works, you're ready to use AI-powered actions.

---

## Your first automation

With either Cloud or local mode set up, you can start automating. Here's the simplest possible example using the Cloud client:

```python
import asyncio
from skyvern import Skyvern

async def main():
    # For local mode, replace this line with: skyvern = Skyvern.local()
    skyvern = Skyvern(api_key="your-api-key")

    # Launch a browser (cloud-hosted on Skyvern's infrastructure)
    browser = await skyvern.launch_cloud_browser()
    page = await browser.get_working_page()

    # Navigate to a site using standard Playwright
    await page.goto("https://news.ycombinator.com")

    # Extract structured data using AI
    top_posts = await page.extract(
        prompt="Get the top 5 post titles and their point counts",
        schema={
            "type": "array",
            "items": {
                "type": "object",
                "properties": {
                    "title": {"type": "string"},
                    "points": {"type": "integer"}
                }
            }
        }
    )
    print(top_posts)

    await browser.close()

asyncio.run(main())
```

`page.extract()` sends the current page to the LLM with your prompt and schema, and returns typed data. The schema is standard JSON Schema, so it works the same way you'd describe a type in OpenAPI.

### Running a task without a browser

If you don't need to drive the browser directly and just want Skyvern to accomplish a goal autonomously, use `run_task()` on the client directly. This is the API-only path that doesn't require you to manage a browser session at all:

```python
import asyncio
from skyvern import Skyvern

async def main():
    skyvern = Skyvern(api_key="your-api-key")

    # Start a task run. Returns immediately with a run_id.
    task = await skyvern.run_task(
        prompt="Go to https://news.ycombinator.com and return the top 5 post titles",
        data_extraction_schema={
            "type": "array",
            "items": {"type": "object", "properties": {"title": {"type": "string"}}}
        },
        max_steps=10,
    )

    print(f"Task started: {task.run_id}")  # run_id starts with "tsk_"

    # Poll until the task finishes
    while True:
        result = await skyvern.get_run(task.run_id)
        print(f"Status: {result.status}")
        if result.status in ("completed", "failed", "terminated", "timed_out", "canceled"):
            break
        await asyncio.sleep(3)

    print(f"Output: {result.output}")

asyncio.run(main())
```

The `status` field cycles through `created → queued → running → completed` (or one of the failure states). The `output` field contains whatever the agent extracted, structured according to your schema.

---

## Mixing Playwright and AI actions

One of the more useful patterns is combining standard Playwright calls with AI-powered actions on the same page. Use Playwright for the deterministic parts (navigation, known selectors), and AI for the parts that vary or require reading the page:

```python
await page.goto("https://myapp.example.com")

# Traditional Playwright: fill a known field
await page.fill("#email", "user@example.com")

# AI-powered click: handles button label variations
await page.click(prompt="Click the 'Continue' or 'Next' button")

# Validate the result
is_logged_in = await page.validate("Check if the user is now logged in")
if not is_logged_in:
    print("Login did not complete successfully")
```

`page.click()` and `page.fill()` also support a **fallback mode** (the default): pass both a CSS selector and a `prompt`. Skyvern tries the selector first, and only calls the LLM if the selector fails. This keeps costs low for pages where the selector is stable, while still handling layout changes gracefully:

```python
# Try the selector; fall back to AI if "#submit-btn" doesn't match
await page.click("#submit-btn", prompt="Click the submit button")
```

---

## Self-hosted with Docker Compose

If you want to run the full Skyvern server (API, UI, database, browser) on your own infrastructure, use Docker Compose.

**Requirements:** Docker Desktop installed and running.

```bash
# Clone the repo
git clone https://github.com/skyvern-ai/skyvern.git && cd skyvern

# Run the quickstart and choose "Docker Compose" when prompted
pip install skyvern && skyvern quickstart
```

The quickstart configures the Compose stack with your LLM keys. Once running, the API is available at `http://localhost:8000` and the UI at `http://localhost:8080`.

To point the Python SDK at your self-hosted instance, use `SkyvernEnvironment.LOCAL`:

```python
from skyvern import Skyvern
from skyvern.client import SkyvernEnvironment

skyvern = Skyvern(
    api_key="your-local-api-key",
    environment=SkyvernEnvironment.LOCAL,  # points to http://localhost:8000
)
```

---

## Next steps

Once you can run the examples above without errors, you're in good shape to go deeper:

- **Tasks**: Use `run_task()` for single-goal automations. Add `data_extraction_schema` for structured output and `max_steps` for cost control.
- **Credentials**: Store login credentials once with `create_credential()`, then have the agent handle authentication automatically with `page.agent.login(credential_id=...)`.
- **Workflows**: Chain multiple steps into a reusable workflow using `run_workflow()`. Workflows are defined in the UI or via the API and identified by a `wpid_` ID.
- **Browser sessions**: Use `launch_cloud_browser()` to open a persistent session you can resume across multiple task runs, useful for sites that need you to stay logged in.

