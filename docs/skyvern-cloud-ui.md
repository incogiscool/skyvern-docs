# Skyvern Cloud UI

The Skyvern UI at [app.skyvern.com](https://app.skyvern.com) is the visual interface for building, running, and monitoring browser automations without writing code. Everything you can do through the REST API or Python SDK is also available here, but presented as a point-and-click experience with live browser streaming, run history, and a visual workflow editor.

This page walks through each section of the UI so you know what's where and how the pieces fit together.

---

## Navigation

The left sidebar is your primary way to move around. It has two groups:

**Build** contains the tools you use to create and monitor automations:
- **Discover**: a starting page with a prompt box and a carousel of pre-built workflow templates
- **Workflows**: your library of saved workflows, with a visual editor for each
- **Runs**: a unified history of every task and workflow run across your account
- **Schedules**: cron-based triggers for running workflows automatically (feature-flagged; appears when enabled)
- **Browsers**: persistent browser sessions you can open, reuse, and attach to runs

**General** contains configuration:
- **Settings**: API keys, environment selection, and integrations (Bitwarden, 1Password, Azure, custom credential services)
- **Credentials**: stored passwords, credit cards, secrets, and 2FA codes

The sidebar collapses to icon-only mode using the chevron button at the bottom, which is useful on smaller screens.

---

## Discover: Your Starting Point

The Discover page (`/discover`) is where most automation work begins. It combines two things:

**The prompt box** sits at the top. You type a plain-English description of what you want a browser to do, optionally provide a starting URL, and click Run. Skyvern generates a workflow from the prompt and immediately starts a run. The default engine is `v2-code`, which generates a reusable script for your task. Advanced settings in the prompt box let you set a proxy location, webhook callback URL, a data extraction schema (as JSON), extra HTTP headers, and whether to save the result as a published workflow.

**Workflow templates** sit below the prompt box as a horizontal carousel. These are globally available pre-built workflows (job applications, invoice downloads, entity lookups, and similar). Clicking a template opens it directly in the workflow editor's debugger view (`/workflows/:workflowPermanentId/build`), where you can inspect and modify it before running.

---

## Tasks: Quick Single-Use Runs

The Tasks section (`/tasks`) is for one-off browser automations. The page shows a prompt box at the top (same as Discover) and below it a toggle between **Run History** and **My Tasks**.

**Run History** shows a paginated list of past task runs from this page. **My Tasks** shows saved task templates you've created manually.

### Creating a task manually

To create a task with explicit fields rather than a prompt, navigate to `/tasks/create/:template` where template can be one of the built-in sample cases (like `finditparts`, `geico`, `hackernews`) or a saved task ID. The form has:

- **URL**: required; the starting page
- **Navigation Goal**: what Skyvern should accomplish on the page
- **Data Extraction Goal**: what information to pull out when done
- **Navigation Payload**: JSON context Skyvern can reference (usernames, form values, etc.)
- **Extracted Information Schema**: a JSON Schema that constrains the shape of extracted data
- **Error Code Mapping**: JSON mapping of failure conditions to custom error codes
- **Proxy Location**: geographic proxy to use (residential US is the default; options include IE, ES, IN, JP, GB, FR, DE, and many others, plus fine-grained city-level targeting)
- **Webhook Callback URL**: where to POST results when the task finishes
- **TOTP Identifier**: links the task to a 2FA credential for sites that require it
- **CDP Address**: connect to a custom browser via Chrome DevTools Protocol

At least one of Navigation Goal or Data Extraction Goal is required.

### Viewing a task run

Once a task is running, its detail page at `/tasks/:taskId` has four tabs:

**Actions**: the main view while a task runs. A live WebSocket stream shows a screenshot updating every few seconds. Once the task finishes, you can click through each action Skyvern took, seeing the screenshot at that moment alongside what was clicked or typed and why. This is where you debug unexpected behavior.

**Recording**: a full video recording of the browser session. Available after the run completes. If no recording was captured, the tab says so rather than failing silently.

**Parameters**: a read-only view of every input field that was sent with this task (URL, goals, payload, schema, proxy, and so on). Useful for reproducing a run or checking what exactly was submitted.

**Diagnostics**: the raw step-by-step artifacts from the agent's internal reasoning. Each step has the full element tree, the LLM prompt, and the model's response. This is useful when a task behaves unexpectedly and you want to understand what the model "saw."

The detail page header shows the current status badge, a cancel button (visible while the task is still running), and options to re-run the task or copy the API command that would reproduce this run.

---

## Workflows: Multi-Step Automations

Workflows chain multiple blocks together into a repeatable process. The Workflows section (`/workflows`) shows your full library in a table with search and filtering. Workflows can be organized into folders, bookmarked, and exported as JSON or YAML.

### The visual editor

Opening a workflow and navigating to **Edit** (`/workflows/:workflowPermanentId/edit`) loads the workflow editor, which is a React Flow canvas. Each workflow is a directed graph of blocks connected by edges.

The editor toolbar (top bar) has:
- A title field you can click to rename the workflow inline
- A save button (with unsaved-changes detection)
- A play button to run the workflow immediately
- A schedule button (clock icon) to open the schedule panel
- A parameters button to open the parameters panel
- A history button to view past versions

The **block library panel** slides in from the right when you click the `+` button. Available block types are:

| Block | What it does |
|-------|-------------|
| Login Block | Authenticates to a website using stored credentials |
| Browser Task Block | Takes a sequence of actions to complete a multi-step goal |
| Browser Action Block | Takes a single discrete action |
| Extraction Block | Extracts structured data from a webpage |
| AI Validation Block | Uses a model to verify the page is in an expected state |
| Human Interaction Block | Pauses the workflow until a human approves or provides input |
| Go to URL Block | Navigates directly to a specific URL |
| Text Prompt Block | Processes text with an LLM (no browser interaction) |
| Conditional Block | Branches execution based on a condition |
| Send Email Block | Sends an email notification |
| Loop Block | Repeats a set of blocks for each item in a list |
| Code Block | Runs custom Python code |
| File Parser Block | Parses PDFs, CSVs, Excel files, and images |

Each block node on the canvas has an edit panel you open by clicking the block. Within that panel you configure goals, data schemas, output variable names, error handling, and references to workflow parameters using Jinja syntax (`{{parameter_key}}`).

### Workflow parameters

Parameters are reusable inputs you define once and reference across blocks. The **parameters panel** (accessible from the header) lists all parameters for the workflow. Parameter types include plain workflow parameters (filled in at run time), AWS Secrets, Bitwarden credentials, 1Password items, Azure Vault credentials, and custom credential service entries.

When a parameter is in scope, you can reference it in any block's text field using `{{parameter_key}}`. The editor shows which blocks reference a given parameter, and deleting a parameter warns you about affected blocks.

### Running a workflow

Clicking the play button in the editor, or clicking **Run** from the workflow list, takes you to a parameters form (`/workflows/:workflowPermanentId/run`). This form shows every workflow-type parameter with a text field. Fill in the values and submit to start a run.

Advanced run options (available from the same form) include proxy location, max screenshot scrolls, webhook callback URL, extra HTTP headers, and run engine selection.

### The debugger view

The debugger (`/workflows/:workflowPermanentId/build`) is a read-alongside-run view that combines the canvas with live output. As blocks execute, their outputs appear inline on the nodes. This is useful for building and iterating on a workflow while watching it run, rather than having to switch between the editor and the run detail page.

### Workflow run detail

Each run at `/workflows/:workflowPermanentId/:workflowRunId` has its own detail view with:

**Overview**: a live browser stream while the workflow runs, switching to a screenshot view afterward. A timeline on the left lists every block and action in execution order. Clicking an item in the timeline jumps the screenshot to that moment.

**Output**: extracted data and downloaded files from the run. If a block produced structured output, it appears here as formatted JSON.

**Code**: if the workflow was run in `code` mode (the default for prompt-generated workflows), this tab shows the generated Python-equivalent script and lets you iterate on it by submitting fix instructions.

**Recording**: a video playback of the full session.

**Parameters**: the input values that were used for this run.

---

## Runs: Unified History

The Runs page (`/runs`) shows every task run and workflow run in a single table, regardless of which section created them. You can filter by status (`created`, `running`, `failed`, `terminated`, `completed`, `queued`, `timed_out`, `canceled`, `paused`) and by trigger type (manual, scheduled, API).

Clicking any row navigates to that run's detail page. The `RunRouter` component at `/runs/:runId/*` automatically redirects to the correct detail view, whether the run is a task or a workflow run.

Search works on both run IDs and parameter values. When a search term matches a workflow parameter, the matching parameter row expands inline without requiring a separate page load.

---

## Schedules

The Schedules page (`/schedules`) lists all cron-based workflow schedules across your organization. Each schedule shows the workflow name, a human-readable description of the cron expression (for example, "Every Monday at 9 AM"), its enabled/paused state, and the last/next run times.

From this page you can create a new schedule, enable or disable individual schedules, duplicate a schedule, and delete one or more. Bulk operations have a concurrency limit built in to avoid overwhelming the API.

You can also manage schedules from within a workflow's editor by clicking the clock icon in the editor toolbar. That opens the schedule panel for that specific workflow, where you create cron schedules and see which are currently active.

---

## Credentials

The Credentials page (`/credentials`) is where you store sensitive values that workflows need to log into sites or fill out forms. It has four tabs:

**Passwords**: stores username/password pairs. Each credential is tested in the background when first saved. You can add, edit, and delete credentials from this tab.

**Credit Cards**: stores card numbers, expiry dates, and billing details for use in checkout flows.

**Secrets**: stores arbitrary secret values (API keys, tokens, etc.) that blocks can reference by name.

**2FA**: shows incoming TOTP codes. When a workflow encounters a 2FA challenge, Skyvern looks up a code here by the `totp_identifier` field. The tab lets you filter codes by identifier and OTP type, and shows when each code was received.

Adding a credential uses a dropdown that opens the appropriate modal (Password, Credit Card, or Secret). Credentials for passwords and credit cards require a Bitwarden-compatible credential service to be configured on the backend.

---

## Browser Sessions

Browser Sessions (`/browser-sessions`) are persistent, reusable browser instances. Unlike a normal task run where a fresh browser starts and closes, a browser session keeps the browser alive between runs. This means cookies, local storage, and login state persist, which is useful for sites that require complex authentication flows or where you want to maintain a logged-in state across multiple workflows.

From this page you can:

- **Create a session**: choose the browser type (Chrome or Microsoft Edge), enable extensions (ad blocker, captcha solver), set a proxy location, and give the session a label
- **View session status**: whether it's open or closed, when it was started, how long it's been running
- **Open a live stream**: click into a session to see a real-time view of the browser and interact with it directly
- **Copy the session ID**: paste it into the CDP address field of a task to attach that task to this session
- **Stop or delete sessions**: from the row actions menu

Sessions are identified by a UUID. Attaching a task to a session passes its `browser_session_id` and the task runs inside that browser's existing context rather than a fresh one.

---

## Settings

The Settings page (`/settings`) has several cards:

**Settings**: select the active environment (local, staging, production) and organization.

**API Key**: shows the currently active API key in a masked copyable field. This is the key you use to authenticate SDK and REST API calls.

**1Password Integration**: enter a 1Password service account token to enable workflows to pull credentials directly from your 1Password vault via workflow parameters.

**Bitwarden Integration**: configure Bitwarden account credentials so workflows can authenticate against a self-hosted or cloud Bitwarden vault.

**Azure Key Vault**: configure an Azure service principal so workflows can read secrets from Azure Key Vault.

**Custom Credential Service**: point Skyvern at a custom HTTP endpoint that returns credentials, for cases where credentials live in an internal system that isn't Bitwarden or Azure.

---

## How the UI Connects to the API

Everything the UI does goes through the same REST API that the Python SDK and direct HTTP clients use. The "Copy API Command" button in task and workflow detail views shows the exact `curl` command or SDK call that would reproduce a given run. This makes it straightforward to start building visually and then migrate to programmatic execution when you need it.

Status values, proxy locations, block types, and parameter references are identical between the UI and the API, so the concepts transfer directly. A workflow you build in the editor exports cleanly to JSON or YAML for version control or import into another environment.
