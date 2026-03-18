# Getting Started with Skyvern

By the end of this guide you'll have Skyvern running locally and will have executed your first AI-powered browser task.

## Prerequisites

- Python 3.11 or 3.12 (3.13 is supported but 3.11/3.12 are most tested)
- Node.js and npm (needed to run the web UI)
- An API key for an LLM provider — OpenAI, Anthropic, and Google Gemini all work

On Windows you also need [Rust](https://rustup.rs/) and Visual Studio with C++ build tools installed.

---

## Option A: pip install (recommended)

This is the fastest path. Skyvern's CLI handles database setup, browser configuration, and environment bootstrapping for you.

### Step 1 — Install Skyvern

```bash
pip install skyvern
```

### Step 2 — Run the quickstart wizard

```bash
skyvern quickstart
```

The wizard will ask you to:

1. Choose a setup method — pick **pip** since you already installed it
2. Select an LLM provider and enter your API key
3. Wait while it installs Playwright browsers and configures a PostgreSQL container

When it finishes, the Skyvern server starts on port 8000 and the web UI opens at [http://localhost:8080](http://localhost:8080).

### Step 3 — Run your first task

Open the UI at [http://localhost:8080](http://localhost:8080), paste a URL into the task form, describe what you want done, and click **Run**. For example:

- URL: `https://news.ycombinator.com`
- Goal: `Get the titles and URLs of the top 5 posts`

Skyvern opens a browser, navigates the page, and returns the extracted data.

---

## Option B: Docker Compose

Use this if you'd rather keep everything containerized or don't want to manage a local Python environment.

### Step 1 — Install Docker Desktop

Download and start [Docker Desktop](https://www.docker.com/products/docker-desktop/).

### Step 2 — Clone the repository

```bash
git clone https://github.com/skyvern-ai/skyvern.git
cd skyvern
```

### Step 3 — Run the quickstart

```bash
pip install skyvern
skyvern quickstart
```

When prompted, choose **Docker Compose**. The wizard pulls the pre-built images (`public.ecr.aws/skyvern/skyvern:latest` and `public.ecr.aws/skyvern/skyvern-ui:latest`), writes a `.env` file with your LLM credentials, and starts three services:

- `postgres` — task and workflow storage (internal, port 5432)
- `skyvern` — API server and browser agent (port 8000)
- `skyvern-ui` — web interface (port 8080)

### Step 4 — Verify it's running

```bash
curl http://localhost:8000/api/v1/heartbeat
```

A `200 OK` response confirms the server is up. Open the UI at [http://localhost:8080](http://localhost:8080).

---

## Option C: From source (for contributors)

### Step 1 — Clone and install

```bash
git clone https://github.com/skyvern-ai/skyvern.git
cd skyvern
pip install -e .
```

### Step 2 — Initialize the environment

```bash
skyvern init
```

This creates your `.env` file and runs database migrations (`alembic upgrade head`).

### Step 3 — Start the services

```bash
skyvern run all
```

This starts the API server on port 8000 and the UI on port 8080.

---

## Using the Python SDK

Once Skyvern is running, you can drive it from Python instead of the UI. The `Skyvern` class connects to your local instance or Skyvern Cloud.

```python
import asyncio
from skyvern import Skyvern

async def main():
    # Connect to your local Skyvern instance.
    # Reads SKYVERN_API_KEY and other settings from .env automatically.
    skyvern = Skyvern.local()

    # Run a task and wait for it to finish before returning.
    result = await skyvern.run_task(
        prompt="Go to news.ycombinator.com and return the titles of the top 5 posts",
        wait_for_completion=True,
    )

    print(result.status)   # "completed"
    print(result.output)   # extracted data as a dict

asyncio.run(main())
```

`Skyvern.local()` requires a `.env` file created by `skyvern quickstart`. To connect to [Skyvern Cloud](https://app.skyvern.com) instead, pass your cloud API key directly:

```python
skyvern = Skyvern(api_key="your-cloud-api-key")
```

---

## Configuring an LLM provider

If you skipped the wizard or need to change providers, edit your `.env` file and restart the server with `skyvern run server`.

**OpenAI (GPT-4o)**
```bash
ENABLE_OPENAI=true
LLM_KEY=OPENAI_GPT4O
OPENAI_API_KEY=sk-...
```

**Anthropic (Claude 3.5 Sonnet)**
```bash
ENABLE_ANTHROPIC=true
LLM_KEY=ANTHROPIC_CLAUDE3.5_SONNET
ANTHROPIC_API_KEY=sk-ant-...
```

**Google Gemini**
```bash
ENABLE_GEMINI=true
LLM_KEY=GEMINI
GEMINI_API_KEY=...
```

The `LLM_KEY` tells Skyvern which configured provider to use as its primary model. All LLM traffic routes through LiteLLM, so the provider switch is transparent to the rest of the system.

---

## Verifying the installation

Run these two commands to confirm everything is wired up:

```bash
# 1. Server health check
curl http://localhost:8000/api/v1/heartbeat

# 2. Submit a test task via the API
curl -X POST http://localhost:8000/api/v1/run/tasks \
  -H "Content-Type: application/json" \
  -H "x-api-key: $(grep SKYVERN_API_KEY .env | cut -d= -f2)" \
  -d '{
    "prompt": "What is the title of the page?",
    "url": "https://example.com"
  }'
```

The second command returns a JSON object with a `run_id`. Poll `GET /api/v1/runs/{run_id}` until `status` is `completed`, then check `output` for the result.

---

## Next steps

- **Architecture** — understand how the agent, LLM, and browser work together
- **SDK Guide** — use `SkyvernBrowserPage` for Playwright-compatible AI automation
- **Workflows** — chain multiple steps into reusable automation templates
- **API Reference** — full endpoint documentation for integrating Skyvern into your stack
