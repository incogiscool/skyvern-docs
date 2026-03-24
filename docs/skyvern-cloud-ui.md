# Skyvern Cloud UI

The Skyvern UI at [app.skyvern.com](https://app.skyvern.com) is the visual interface for building, running, and monitoring browser automations. It covers everything the REST API and Python SDK can do — live browser streaming, a visual workflow editor, run history, credential storage, and scheduled triggers — all without writing code. It's also the fastest way to prototype: describe a task in plain English, click Run, and watch it execute in seconds.

This page walks through each section of the UI so you know what's where and how the pieces fit together.

---

## On this page

- [How to navigate the UI](#navigation)
- [Discover — run your first automation](#discover-your-starting-point)
- [Tasks — quick single-use runs](#tasks-quick-single-use-runs)
- [Workflows — multi-step automations](#workflows-multi-step-automations)
- [Runs — unified history](#runs-unified-history)
- [Schedules — cron triggers](#schedules)
- [Credentials — passwords, cards, and secrets](#credentials)
- [Browser Sessions — persistent browsers](#browser-sessions)
- [Settings — API keys and integrations](#settings)
- [How the UI connects to the API](#how-the-ui-connects-to-the-api)

---

## Navigation

The left sidebar is your primary way to move around. It has two groups.

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

The Discover page (`/discover`) is where most automation work begins. Type what you want a browser to do, hit Run, and Skyvern handles the rest — no workflow design required upfront.

**The prompt box** sits at the top. You type a plain-English description of what you want a browser to do, optionally provide a starting URL, and click Run. Skyvern generates a workflow from the prompt and immediately starts a run. The default engine is `v2-code`, which generates a reusable script for your task. Advanced settings in the prompt box let you set a proxy location, webhook callback URL, a data extraction schema (as JSON), extra HTTP headers, and whether to save the result as a published workflow.

**Workflow templates** sit below the prompt box as a horizontal carousel. These are globally available pre-built workflows (job applications, invoice downloads, entity lookups, and similar). Clicking a template opens it directly in the workflow editor's debugger view (`/workflows/:workflowPermanentId/build`), where you can inspect and modify it before running.

---

## Tasks: Quick Single-Use Runs

The Tasks section (`/tasks`) is for one-off browser automations. The page shows a prompt box at the top (same as Discover) and below it a toggle between **Run History** and **My Tasks**. Run History shows a paginated list of past task runs from this page; My Tasks shows saved task templates you've created manually.

### Creating a task manually

To create a task with explicit fields rather than a prompt, navigate to `/tasks/create/:template`, where `template` can be one of the built-in sample cases (like `finditparts`, `geico`, `hackernews`) or a saved task ID. At least one of Navigation Goal or Data Extraction Goal is required; all other fields are optional.

**What the browser should do:**
- **URL** *(required)* — the starting page
- **Navigation Goal** — what Skyvern should accomplish on the page
- **Data Extraction Goal** — what information to pull out when done
- **Navigation Payload** — JSON context Skyvern can reference (usernames, form values, etc.)
- **Extracted Information Schema** — a JSON Schema that constrains the shape of extracted data
- **Error Code Mapping** — JSON mapping of failure conditions to custom error codes

**How the browser connects:**
- **Proxy Location** — geographic proxy to use (residential US is the default; options include IE, ES, IN, JP, GB, FR, DE, and many others, plus fine-grained city-level targeting)
- **TOTP Identifier** — links the task to a 2FA credential for sites that require it
- **CDP Address** — connect to a custom browser via Chrome DevTools Protocol

**Where results go:**
- **Webhook Callback URL** — where to POST results when the task finishes

### Viewing a task run

Once a task is running, its detail page at `/tasks/:taskId` has four tabs. The header shows the current status badge, a cancel button (visible while the task is still running), and options to re-run the task or copy the `curl` command that would reproduce this run.

**Actions** is the main view while a task runs. A live WebSocket stream shows a screenshot updating every few seconds. Once the task finishes, you can click through each action Skyvern took, seeing the screenshot at that moment alongside what was clicked or typed and why. This is where you debug unexpected behavior.

**Recording** is a full video recording of the browser session, available after the run completes. If no recording was captured, the tab says so rather than failing silently.

**Parameters** is a read-only view of every input field that was sent with this task (URL, goals, payload, schema, proxy, and so on). Useful for reproducing a run or confirming exactly what was submitted.

**Diagnostics** contains the raw step-by-step artifacts from the agent's internal reasoning. Each step has the full element tree, the LLM prompt, and the model's response — the place to go when a task behaves unexpectedly and you want to understand what the model "saw."

---

## Workflows: Multi-Step Automations

Workflows chain multiple blocks together into a repeatable, parameterized process. The Workflows section (`/workflows`) shows your full library in a searchable, filterable table. Workflows can be organized into folders, bookmarked, and exported as JSON or YAML.

### Building a workflow in the visual editor

Opening a workflow and navigating to **Edit** (`/workflows/:workflowPermanentId/edit`) loads the workflow editor — a canvas where each workflow is a directed graph of blocks connected by edges.

The editor toolbar at the top contains:
- A title field you can click to rename the workflow inline
- A save button with unsaved-changes detection
- A play button to run the workflow immediately
- A clock icon to open the schedule panel
- A parameters button to open the parameters panel
- A history button to view past versions

Click the `+` button to open the **block library panel** and add a block to the canvas. Available block types are:

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
| File Download Block | Downloads files from a website |
| Cloud Storage Block | Uploads files to cloud storage |
| Wait Block | Pauses execution for a specified amount of time |
| HTTP Request Block | Makes an HTTP API call |
| Print Page Block | Prints the current page to PDF |
| Workflow Trigger Block | Triggers another workflow |

Click any block node on the canvas to open its edit panel, where you configure goals, data schemas, output variable names, and error handling. Reference workflow parameters in any text field using Jinja syntax — for example, `{{parameter_key}}`.

### Defining and using workflow parameters

Parameters are reusable inputs you define once and reference across any block. Open the **parameters panel** from the header to list, add, or remove parameters for the workflow. Parameter types include plain workflow parameters (filled in at run time), AWS Secrets, Bitwarden credentials, 1Password items, Azure Vault credentials, and custom credential service entries.

The editor shows which blocks reference a given parameter. Deleting a parameter warns you about every affected block before proceeding.

### Running a workflow

Clicking the play button in the editor, or clicking **Run** from the workflow list, opens a parameters form at `/workflows/:workflowPermanentId/run`. Fill in a value for each workflow-type parameter and submit to start a run. Advanced options on the same form include proxy location, max screenshot scrolls, webhook callback URL, and extra HTTP headers.

### Iterating live with the debugger

The debugger (`/workflows/:workflowPermanentId/build`) combines the canvas with live output so you can watch a workflow execute without leaving the editor. As blocks run, their outputs appear inline on the nodes. This is the fastest way to iterate on a workflow: submit a run, see exactly which block produced what output, and edit immediately.

### Inspecting a workflow run

Each run at `/workflows/:workflowPermanentId/:workflowRunId` has its own detail view with five tabs.

**Overview** shows a live browser stream while the workflow runs, then switches to a screenshot view afterward. A timeline on the left lists every block and action in execution order; clicking any item jumps the screenshot to that moment.

**Output** shows extracted data and downloaded files from the run. Structured block output appears here as formatted JSON.

**Code** is available when the workflow ran in `code` mode (the default for prompt-generated workflows). It shows the generated Python-equivalent script and lets you iterate by submitting fix instructions.

**Recording** is a full video playback of the browser session.

**Parameters** shows the exact input values used for this run — useful when you need to reproduce or audit a specific execution.

---

## Runs: Unified History

The Runs page (`/runs`) shows every task run and workflow run in a single table, regardless of which section created them. This is the fastest way to get a cross-account view of what's running, what's failed, and what's queued.

You can filter by status — `created`, `running`, `failed`, `terminated`, `completed`, `queued`, `timed_out`, or `canceled` — and by trigger type (manual, scheduled, or API). Clicking any row navigates to that run's detail page, whether the run is a task or a workflow run.

Search works on both run IDs and parameter values. When a search term matches a workflow parameter, the matching parameter row expands inline without requiring a separate page load.

---

## Schedules

The Schedules page (`/schedules`) lists all cron-based workflow schedules across your organization. Each row shows the workflow name, a human-readable description of the cron expression (for example, "Every Monday at 9 AM"), the enabled/paused state, and the last and next run times.

From this page you can create a new schedule, enable or disable individual schedules, duplicate a schedule, and delete one or more. Bulk operations have a built-in concurrency limit to avoid overwhelming the API.

You can also manage schedules from within any workflow's editor by clicking the clock icon in the toolbar. That opens the schedule panel scoped to that workflow, where you can create cron schedules and see which are currently active.

---

## Credentials

The Credentials page (`/credentials`) is where you store sensitive values that workflows need to log into sites or fill out forms. It has four tabs.

**Passwords** stores username/password pairs. Each credential is tested in the background when first saved. You can add, edit, and delete credentials from this tab. Note that password and credit card credentials require a Bitwarden-compatible credential service to be configured in Settings.

**Credit Cards** stores card numbers, expiry dates, and billing details for use in checkout flows.

**Secrets** stores arbitrary secret values — API keys, tokens, and similar — that blocks can reference by name.

**2FA** shows incoming TOTP codes. When a workflow encounters a 2FA challenge, Skyvern looks up the matching code by the `totp_identifier` field you supply on a task or workflow parameter. The tab lets you filter codes by identifier and OTP type, and shows when each code was received.

---

## Browser Sessions

Browser Sessions (`/browser-sessions`) are persistent, reusable browser instances. Unlike a normal task run — where a fresh browser starts and closes when the task finishes — a browser session keeps the browser alive between runs. Cookies, local storage, and login state all persist, which is particularly useful for sites with complex authentication flows or when you want to stay logged in across multiple workflows.

To create a session, choose the browser type (Chrome or Microsoft Edge), optionally enable extensions (ad blocker, captcha solver), set a proxy location, and give the session a label. Once the session is open, you can click into it for a real-time view of the browser and interact with it directly.

Sessions are identified by a UUID. To attach a run to a session from the Discover or Tasks prompt box, copy the session ID and paste it into the **Browser Session ID** field in the Advanced Settings panel. (The separate **Browser Address** field is for connecting to an arbitrary CDP-compatible browser endpoint, not a Skyvern-managed session.) The task then runs inside that browser's existing context — with all its cookies and saved state — rather than starting fresh. You can stop or delete sessions at any time from the row actions menu.

---

## Settings: Configure API Keys and Integrations

The Settings page (`/settings`) is where you connect Skyvern to external credential stores and manage your API access. Each card on the page corresponds to an integration.

**Settings** lets you select the active environment and organization.

**API Key** shows the currently active API key in a masked, copyable field. This is the key you use to authenticate SDK and REST API calls.

**1Password Integration** accepts a 1Password service account token, which lets workflows pull credentials directly from your 1Password vault via workflow parameters.

**Bitwarden Integration** takes your Bitwarden account credentials and connects Skyvern to a self-hosted or cloud Bitwarden vault — required if you want to use the Passwords or Credit Cards tabs in Credentials.

**Azure Key Vault** takes an Azure service principal configuration so workflows can read secrets from Azure Key Vault.

**Custom Credential Service** accepts a custom HTTP endpoint URL. Skyvern will call that endpoint to retrieve credentials, which covers cases where your credentials live in an internal system that isn't Bitwarden or Azure.

---

## How the UI Connects to the API

Everything the UI does goes through the same REST API that the Python SDK and direct HTTP clients use. The **Copy API Command** button in task and workflow detail views shows the exact `curl` command or SDK call that would reproduce a given run. This makes it straightforward to start building visually and then migrate to programmatic execution when you need it.

Status values, proxy locations, block types, and parameter references are identical between the UI and the API, so the concepts transfer directly. A workflow you build in the editor exports cleanly to JSON or YAML for version control or import into another environment.
