# Getting Started

By the end of this guide, you'll have Skyvern running locally and will have executed your first browser automation task - either through the web UI or the Python SDK.

## Prerequisites

- Python 3.11 or 3.12 (3.13 is not yet supported)
- Node.js and npm (required for the frontend UI)
- 4 GB of free RAM (the browser + LLM calls are memory-hungry)
- An API key from at least one LLM provider: OpenAI, Anthropic, Google Gemini, or another supported provider

Windows users also need [Rust](https://rustup.rs/) and VS Code with C++ dev tools and the Windows SDK installed.

## Option A: pip install (recommended)

This is the fastest path to a working local setup.

**1. Install the package:**

```bash
pip install skyvern
```

**2. Run the setup wizard:**

```bash
skyvern quickstart
```

The wizard walks you through choosing an LLM provider, entering your API key, and starting the server. When it finishes, the backend is running on `http://localhost:8000` and the UI is at `http://localhost:8080`.

## Option B: Docker Compose

If you prefer a fully containerized setup (useful for clean environments or CI), use Docker Compose instead.

**1. Install [Docker Desktop](https://www.docker.com/products/docker-desktop/) and start it.**

**2. Clone the repository:**

```bash
git clone https://github.com/skyvern-ai/skyvern.git
cd skyvern
```

**3. Run the quickstart:**

```bash
pip install skyvern && skyvern quickstart
```

When prompted, choose "Docker Compose." This starts PostgreSQL, the Skyvern backend on port 8000, and the UI on port 8080 - all containerized.

**4. Open the UI:** Navigate to `http://localhost:8080`.

## Your first task via the UI

Once Skyvern is running, open `http://localhost:8080`. You'll see the task runner interface. Paste a goal into the prompt field - something like:

```
Navigate to https://news.ycombinator.com and return the titles of the top 3 posts.
```

Click **Run**. Skyvern opens a browser window, takes a screenshot, sends it to the LLM, and starts clicking. You can watch it work in real time. When the task completes, the extracted data appears in the results panel on the right.

## Your first task via the Python SDK

If you'd rather automate programmatically, the Python SDK is the most direct path.

**1. Create a small script:**

```python
import asyncio
from skyvern import Skyvern

async def main():
    skyvern = Skyvern()  # connects to your local server
    task = await skyvern.run_task(
        prompt="Navigate to https://news.ycombinator.com and return the titles of the top 3 posts."
    )
    print(task)

asyncio.run(main())
```

**2. Run it:**

```bash
python main.py
```

`run_task` blocks until the task is complete and returns the result. You should see the extracted post titles printed to your terminal.

If you want a structured response instead of free-form text, pass a JSON schema:

```python
task = await skyvern.run_task(
    prompt="Get the top 3 posts on Hacker News.",
    data_extraction_schema={
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
```

Skyvern will return data that conforms to the schema, which you can parse and use directly.

## Verifying everything works

After running your first task, check these signals:

- The task status should be `completed` (not `failed` or `timed_out`).
- The result contains the data you asked for.
- The UI at `http://localhost:8080` shows the task in your run history, with screenshots and a recording you can replay.

If you see `failed` or a blank result, the most common cause is a missing or invalid LLM API key. Re-run `skyvern quickstart` and double-check the key you entered.

## Using Skyvern Cloud instead

If you don't want to manage local infrastructure, [Skyvern Cloud](https://app.skyvern.com) is a managed version with the same API. Create an account, grab your API key from Settings, and point the SDK at the cloud:

```python
skyvern = Skyvern(api_key="your-cloud-api-key")
```

Everything else - task creation, result retrieval, workflows - works the same way.

## Next steps

- **Architecture** - learn how the Vision LLM loop works under the hood
- **Workflows** - chain multiple tasks together with conditional logic
- **Python SDK** - full reference for the `Skyvern` client
- **Credentials & Auth** - store passwords and API keys for use in automation tasks
