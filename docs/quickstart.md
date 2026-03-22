# Quickstart

Run your first browser automation task with Skyvern in under five minutes.

## Prerequisites

- Python 3.11, 3.12, or 3.13
- A Skyvern Cloud account and API key - sign up at [app.skyvern.com](https://app.skyvern.com) (includes $5 in free credits)

## Steps

### 1. Install the SDK

```bash
pip install skyvern
```

That's the only dependency you need to start running tasks against Skyvern Cloud. The SDK ships with a synchronous `Skyvern` client and an `AsyncSkyvern` client for use in async codebases.

### 2. Get your API key

Log into [app.skyvern.com](https://app.skyvern.com), navigate to **Settings**, and copy your API key.

### 3. Run a task

Paste the snippet below into a file called `run_task.py`, replace `YOUR_API_KEY`, and run it.

```python
import time
from skyvern import Skyvern

client = Skyvern(api_key="YOUR_API_KEY")

# Submit the task - the browser runs on Skyvern's servers by default
run = client.run_task(
    url="https://news.ycombinator.com",
    prompt="Find the top 3 posts on Hacker News today and return their titles and URLs.",
)
print(f"Task started: {run.run_id}  (status: {run.status})")

# Poll until the run finishes
while run.status in ("created", "queued", "running"):
    time.sleep(5)
    run = client.get_run(run.run_id)
    print(f"  status: {run.status}")

print(f"\nFinal status: {run.status}")
print(f"Output: {run.output}")
print(f"View in UI: {run.app_url}")
```

```bash
python run_task.py
```

`run_task()` returns immediately with a `run_id` (prefixed `tsk_`) and an initial status of `created` or `queued`. The task runs asynchronously on Skyvern's infrastructure, so you poll `get_run()` every few seconds until the status moves out of the active set (`created`, `queued`, `running`) and settles on `completed`, `failed`, or another terminal state.

### 4. See the results and watch the recording

Once the task is `completed`, `run.output` holds whatever the agent extracted. Open `run.app_url` in your browser to see the full picture inside the Skyvern UI:

- **Actions** - every step the agent took, each paired with a screenshot of the browser state after that action
- **Recording** - a full video of the session from start to finish
- **Parameters** - the prompt and URL you submitted
- **Diagnostics** - annotated screenshots, the element tree, and the raw LLM requests, which are useful when the agent goes in an unexpected direction

You can also pull the recording URL directly from the SDK if you want to store or display it in your own tooling:

```python
artifacts = client.get_run_artifacts(run.run_id, artifact_type="recording")
if artifacts:
    print(f"Recording: {artifacts[0].uri}")
```

## Optional: run with a local browser instead

By default, Skyvern spins up a managed browser on its own servers. If you want to watch the automation happen live on your machine, or need to reach a site that is only accessible from your network, you can point the Skyvern Cloud API at a browser running locally.

Start the local browser server in a separate terminal:

```bash
skyvern browser serve --tunnel
```

This launches Chrome on your machine, starts a CDP proxy server, and creates an ngrok tunnel so Skyvern Cloud can connect to it. When it's ready, the command prints a `wss://` URL.

Then pass that URL as `browser_address` in your task call. Everything else stays the same:

```python
# Cloud browser (default) - no extra arguments needed
run = client.run_task(
    url="https://news.ycombinator.com",
    prompt="Find the top 3 posts on Hacker News today.",
)

# Local browser - add browser_address with the tunnel URL from `skyvern browser serve --tunnel`
run = client.run_task(
    url="https://news.ycombinator.com",
    prompt="Find the top 3 posts on Hacker News today.",
    browser_address="wss://abc123.ngrok-free.dev",
)
```

The polling, results, and recording all work identically. The only difference is where the browser process runs.

> `--tunnel` requires ngrok to be installed and authenticated (`ngrok authtoken <your-token>`). See the [Connect Local Browser](../optimization/browser-tunneling) guide for full setup details, security notes, and how to configure a manual tunnel.

## What you verified

After completing these steps you have:

- A `run_id` (prefixed `tsk_`) confirming the task was accepted by Skyvern Cloud
- A final status of `completed` with extracted output in `run.output`
- A link to the Skyvern UI where you can replay the full browser recording

## Next steps

- **Structured output** - add a `data_extraction_schema` (JSON Schema) to `run_task()` to get typed, predictable data back instead of free-form text
- **Authentication** - store credentials with `create_credential()` and reference them in your prompt so Skyvern can log in on your behalf
- **Multi-step automation** - use Workflows to chain tasks with loops, conditionals, and file handling
- **Async notification** - pass a `webhook_url` to `run_task()` to receive a callback when the run finishes, instead of polling
