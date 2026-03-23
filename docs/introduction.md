# Skyvern

Skyvern automates browser-based workflows using LLMs and computer vision. Instead of writing scripts that click specific DOM elements or XPath selectors, you describe what you want in plain language and Skyvern figures out how to do it on any website, even ones it has never seen before.

## Why Skyvern doesn't break when websites change

Traditional browser automation is fragile. A script that clicks `div#submit-button > span.text` breaks the moment a designer renames a class or restructures a component. You end up in an endless cycle of maintaining selectors that drift out of sync with the sites you depend on.

Skyvern sidesteps this entirely. At each step, it takes a screenshot of the current browser viewport, sends it to a vision-capable LLM along with your goal, and receives back a plan of concrete actions (click, type, scroll, select, and so on). Because the model reasons from what it actually *sees*, layout changes that would break a selector-based script often have no effect on Skyvern at all.

This also means Skyvern can work across multiple websites with a single workflow. If your goal is "find the cheapest shipping option," that same instruction applies to dozens of different carrier sites without any site-specific code.

## What you can automate

Skyvern works on the kind of tasks that normally require a human sitting at a browser:

- **Form submission**: job applications, insurance quotes, government forms, account registrations
- **Data extraction**: pulling product prices, scraping search results, reading invoices
- **Transactional flows**: adding items to a cart, completing a checkout, downloading receipts
- **Authenticated workflows**: logging in, navigating a portal, handling 2FA/TOTP, then completing a task

## Choose your starting point

There are three ways to use Skyvern, and the right one depends on where your browser should run.

### Skyvern Cloud (no setup required)

Go to [app.skyvern.com](https://app.skyvern.com), create an account, and run your first task from the web UI in minutes. The Discover page has ready-made templates for common automations. The browser runs on Skyvern's servers, with built-in proxies and anti-bot detection included.

This is the right starting point for most people.

### Python SDK and REST API

If you're integrating Skyvern into an application or pipeline, install the SDK and call it directly:

```bash
pip install skyvern
```

Then run a task against Skyvern Cloud with your API key from [app.skyvern.com/settings](https://app.skyvern.com/settings):

```python
import asyncio
from skyvern import Skyvern

client = Skyvern(api_key="YOUR_API_KEY")

task = asyncio.run(
    client.run_task(
        prompt="Find the top 3 posts on Hacker News today.",
        url="https://news.ycombinator.com",
        wait_for_completion=True,  # bool, default False — blocks until task reaches a final status
    )
)

print(task.status)  # "completed"
print(task.output)  # {"answer": "1. ... 2. ... 3. ..."}
```

`run_task` accepts a natural language `prompt` describing your goal, an optional starting `url`, and a handful of parameters that control output shape, cost limits, and authentication. The returned object exposes `task.status` (one of `"completed"`, `"failed"`, `"timed_out"`, `"terminated"`, or `"canceled"`) and `task.output` (the extracted data as a dict, list, or string).

If you want the browser to appear on your own machine instead — useful for debugging, accessing internal tools, or watching the automation live — run `skyvern quickstart` first to configure a local server, then point the client at `http://localhost:8000`.

### Self-hosted

For full data privacy or high-volume use cases, you can run the entire stack on your own infrastructure. The Docker Compose setup brings up the Skyvern server, a Playwright-powered browser, and a PostgreSQL database. You supply your own LLM API keys (OpenAI, Anthropic, Azure, Gemini, and others are all supported via [LiteLLM](https://github.com/BerriAI/litellm), a unified routing layer for LLM providers).

## Core concepts

Understanding three concepts covers the majority of what Skyvern does.

**Tasks** are the atomic unit of work — a single goal expressed in plain language. Give Skyvern an instruction like "log in and download this month's invoice," and it executes the goal step by step and returns a result, plus an optional recording of everything that happened.

**Workflows** let you chain tasks together with logic between them. A workflow is a YAML-defined sequence of blocks: a `task` block navigates a website, a `for_loop` block iterates over a list of inputs, a `conditional` block branches based on extracted data, and a `code` block runs arbitrary Python. Reach for workflows when a single task isn't enough — for example, logging in, paginating through a report, and emailing the results.

**Credentials** let Skyvern handle authentication without you embedding passwords in prompts. You store a username/password pair (or credit card, or TOTP secret) in the Skyvern credential vault and reference it by name in a task or workflow. Skyvern fills in the credentials at runtime and handles 2FA automatically.

## Choosing an engine

The `engine` parameter (type: `str`, default: `"skyvern-2.0"`) controls how Skyvern approaches navigation:

- `skyvern-2.0` — the current flagship agent, best for complex multi-step tasks. Scores 85.8% on the WebVoyager benchmark.
- `skyvern-1.0` — lower cost, well-suited to simple single-goal tasks like filling a form or pulling a search result.
- `openai-cua` — delegates navigation directly to OpenAI's computer-use model.
- `anthropic-cua` — delegates navigation directly to Anthropic's computer-use model.

## Integrations

Beyond the SDK, Skyvern connects to the tools your team already uses:

- **MCP server**: AI coding assistants like Claude, Cursor, and Windsurf can control a browser directly through the Model Context Protocol, with 39 exposed tools covering navigation, extraction, and credential management.
- **CLI**: `skyvern workflow run` and `skyvern tasks` let you trigger automations from the terminal or CI pipelines. (`skyvern run` starts services — the server, UI, and MCP server.)
- **No-code platforms**: native integrations for Zapier, Make, n8n, and Workato are available for teams that prefer workflow automation tools.

## Where to go next

Start with the Quickstart to get a task running in under five minutes, then explore the reference docs as your needs grow:

- **[Quickstart](quickstart.md)**: get a task running on Skyvern Cloud or locally.
- **[Run Tasks](running-tasks/run-tasks.md)**: full reference for task parameters, data extraction schemas, webhooks, and proxy configuration.
- **[Workflows](workflows/manage-workflows.md)**: how to build, run, and manage multi-step automations.
- **[Credentials](credentials/overview.md)**: storing and using passwords, credit cards, and TOTP codes.
