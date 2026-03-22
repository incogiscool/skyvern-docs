# Page Proposal

This file maps every documentation page that needs to be created or updated, derived from the DOC-PLAN and the existing fern/ content inventory.

## Legend
- **Status**: `exists` (file already in fern/), `new` (needs to be created), `update` (exists but needs revision per DOC-PLAN)
- **Type**: `conceptual` (explains what/why), `how-to` (task-oriented steps), `reference` (API/parameter lookup), `tutorial` (end-to-end walkthrough)
- **Outcome**: One sentence describing what a reader can do after reading the page.

---

## Tab 1: Home (API/SDK & Self-Hosted)

### Getting Started

| # | Page | Output Path | Status | Type | Outcome |
|---|------|-------------|--------|------|---------|
| 1 | Introduction | `fern/introduction.mdx` | exists | conceptual | Reader understands what Skyvern is, how it works, and which deployment mode fits their needs. |
| 2 | Quickstart | `fern/getting-started/quickstart.mdx` | exists | tutorial | Reader can run their first automated browser task using Skyvern Cloud or a local install within minutes. |
| 3 | Skyvern In Action | `fern/getting-started/skyvern-in-action.mdx` | exists | conceptual | Reader can see concrete real-world examples of what Skyvern automates and evaluate fit for their use case. |
| 4 | Prompting and Troubleshooting Guide | `fern/getting-started/prompting-guide.mdx` | exists | how-to | Reader can write effective task prompts and diagnose common failures when Skyvern doesn't behave as expected. |

### Tasks

| # | Page | Output Path | Status | Type | Outcome |
|---|------|-------------|--------|------|---------|
| 5 | Run Tasks | `fern/running-tasks/run-tasks.mdx` | exists | reference | Reader can configure and submit a task with any combination of engine, schema, proxy, TOTP, webhook, and browser options. |
| 6 | Visualizing Results | `fern/running-tasks/visualizing-results.mdx` | exists | how-to | Reader can navigate the run detail view to inspect actions, watch the recording, and read diagnostic logs. |
| 7 | Cancel Task Runs | `fern/running-tasks/cancel-runs.mdx` | exists | how-to | Reader can cancel an in-progress task run via the UI or API. |
| 8 | Webhooks FAQ | `fern/running-tasks/webhooks-faq.mdx` | exists | reference | Reader can set up, validate, and replay webhooks to receive task completion notifications in their system. |
| 9 | Connect Your Local Browser | `fern/running-tasks/connect-local-browser.mdx` | exists | how-to | Reader can point Skyvern Cloud at a browser running on their machine via CDP and an optional ngrok tunnel. |
| 10 | Proxy & Geo Targeting | `fern/running-tasks/proxy-location.mdx` | exists | reference | Reader can route task traffic through a specific country or city by passing the correct proxy_location value. |

### Workflows

| # | Page | Output Path | Status | Type | Outcome |
|---|------|-------------|--------|------|---------|
| 11 | Manage Workflows | `fern/workflows/manage-workflows.mdx` | exists | how-to | Reader can create, edit, clone, export, import, and delete workflows in the Skyvern UI. |
| 12 | Workflow Blocks | `fern/workflows/workflow-blocks-details.mdx` | exists | reference | Reader can identify the right block type for their use case and understand what each of the 25 block types does. |
| 13 | Workflow Parameters | `fern/workflows/workflow-parameters.mdx` | exists | conceptual | Reader can define workflow parameters, reference them with Jinja syntax, and understand reserved parameter names. |
| 14 | Run Workflows | `fern/workflows/run-workflows.mdx` | exists | how-to | Reader can trigger a workflow run via UI or API and supply runtime parameter overrides including proxy location. |
| 15 | Consistency and Reliability | `fern/workflows/consistent-workflows.mdx` | exists | conceptual | Reader understands how Skyvern's script caching and AI fallback mechanisms improve workflow reliability over time. |

### Credentials

| # | Page | Output Path | Status | Type | Outcome |
|---|------|-------------|--------|------|---------|
| 16 | Overview | `fern/credentials/introduction.mdx` | exists | conceptual | Reader understands Skyvern's credential security model and which credential types and password managers are supported. |
| 17 | Password Management | `fern/credentials/passwords.mdx` | exists | how-to | Reader can store a username/password credential and use it in a Login block without exposing it to an LLM. |
| 18 | Credit Card Management | `fern/credentials/credit-cards.mdx` | exists | how-to | Reader can store a credit card credential and use it to complete purchase workflows. |
| 19 | 2FA Support (TOTP) | `fern/credentials/totp.mdx` | exists | how-to | Reader can configure any of the five supported 2FA methods so Skyvern can log into sites that require MFA. |
| 20 | Bitwarden | `fern/credentials/bitwarden.mdx` | exists | how-to | Reader can connect their Bitwarden vault so Skyvern reads credentials on demand without storing them. |
| 21 | Custom Credential Service | `fern/credentials/custom-credential-service.mdx` | exists | how-to | Reader can integrate their own HTTP credential API so Skyvern fetches secrets from their existing secret store. |

### Browser Sessions (Beta)

| # | Page | Output Path | Status | Type | Outcome |
|---|------|-------------|--------|------|---------|
| 22 | Introduction | `fern/browser-sessions/introduction.mdx` | exists | how-to | Reader can create, reuse, and close a persistent browser session to chain multiple tasks without re-authenticating. |
| 23 | Browser Profiles | `fern/browser-sessions/browser-profiles.mdx` | exists | how-to | Reader can snapshot an authenticated browser state into a reusable profile and attach it to future workflow runs. |

### Observability

| # | Page | Output Path | Status | Type | Outcome |
|---|------|-------------|--------|------|---------|
| 24 | Overview | `fern/observability/overview.mdx` | exists | how-to | Reader can instrument Skyvern with Laminar to trace LLM calls, browser recordings, and workflow steps end-to-end. |

### Integrations

| # | Page | Output Path | Status | Type | Outcome |
|---|------|-------------|--------|------|---------|
| 25 | MCP Server | `fern/integrations/mcp.mdx` | exists | how-to | Reader can configure Skyvern's MCP server in their AI assistant (Claude, Cursor, Windsurf) and control a browser via natural language. |
| 26 | CLI & Skills | `fern/integrations/cli.mdx` | exists | reference | Reader can use the `skyvern` CLI to automate browsers, manage workflows and credentials, and configure MCP from the terminal. |
| 27 | Zapier | `fern/integrations/zapier.mdx` | exists | how-to | Reader can trigger Skyvern tasks from a Zapier zap and use the results in downstream automation steps. |
| 28 | Make | `fern/integrations/make.com.mdx` | exists | how-to | Reader can integrate Skyvern into a Make.com scenario to automate browser tasks as part of a larger workflow. |
| 29 | N8N | `fern/integrations/n8n.mdx` | exists | how-to | Reader can add a Skyvern node to an n8n workflow to run browser tasks as part of a multi-step automation. |
| 30 | Workato | `fern/integrations/workato.mdx` | exists | how-to | Reader can connect Skyvern to Workato recipes to automate browser-based steps in enterprise workflows. |
| 31 | Ollama + LiteLLM | `fern/integrations/ollama-litellm.mdx` | exists | how-to | Reader can configure Skyvern to use a locally-running Ollama model via LiteLLM instead of a cloud LLM provider. |

---

## Tab 2: Skyvern Cloud UI

All pages below are **new** (not yet in fern/).

### Getting Started (UI)

| # | Page | Proposed Path | Type | Outcome |
|---|------|--------------|------|---------|
| 32 | UI Overview | `fern/cloud-ui/getting-started/overview.mdx` | conceptual | Reader understands the main sections of the Skyvern Cloud dashboard and when to use the UI versus the API/SDK. |
| 33 | Your First Task | `fern/cloud-ui/getting-started/first-task.mdx` | tutorial | Reader can submit and watch their first task run entirely within the Skyvern Cloud UI. |

### Running Tasks (UI)

| # | Page | Proposed Path | Type | Outcome |
|---|------|--------------|------|---------|
| 34 | The Discover Page | `fern/cloud-ui/tasks/discover-page.mdx` | how-to | Reader can fill in the task creation form on the Discover page and launch an ad-hoc task with advanced settings. |
| 35 | Watching Live Execution | `fern/cloud-ui/tasks/live-execution.mdx` | how-to | Reader can monitor a running task in the live browser view and stop it if needed. |

### Building Workflows (UI)

| # | Page | Proposed Path | Type | Outcome |
|---|------|--------------|------|---------|
| 36 | Workflows List | `fern/cloud-ui/workflows/workflows-list.mdx` | how-to | Reader can view, create, clone, delete, and import/export workflows from the Workflows list page. |
| 37 | The Workflow Editor | `fern/cloud-ui/workflows/workflow-editor.mdx` | how-to | Reader can add, configure, reorder, and delete blocks in the visual workflow editor and save their changes. |
| 38 | Working with Parameters | `fern/cloud-ui/workflows/parameters.mdx` | how-to | Reader can create workflow parameters, reference them in blocks using Jinja syntax, and supply values at run time. |
| 39 | Block Configuration | `fern/cloud-ui/workflows/block-configuration.mdx` | reference | Reader can configure any of the 25 workflow block types correctly through the editor's side panel. |
| 40 | Running Workflows (UI) | `fern/cloud-ui/workflows/running-workflows.mdx` | how-to | Reader can trigger a workflow run, fill in parameter values, select credentials, and monitor the queued status. |

### Viewing Results (UI)

| # | Page | Proposed Path | Type | Outcome |
|---|------|--------------|------|---------|
| 41 | Run History | `fern/cloud-ui/results/run-history.mdx` | how-to | Reader can find past runs in the History page by filtering on status, date, or type and click into any run. |
| 42 | Run Details | `fern/cloud-ui/results/run-details.mdx` | how-to | Reader can interpret the Actions, Recording, Parameters, and Output tabs of a completed run. |
| 43 | Diagnostic Logs | `fern/cloud-ui/results/diagnostic-logs.mdx` | how-to | Reader can use the Diagnostics tab to inspect annotated screenshots, the element tree, and raw LLM requests for debugging. |
| 44 | Downloading Artifacts | `fern/cloud-ui/results/artifacts.mdx` | how-to | Reader can download the recording, screenshots, and HAR file produced by any run. |

### Managing Credentials (UI)

| # | Page | Proposed Path | Type | Outcome |
|---|------|--------------|------|---------|
| 45 | Credentials Overview (UI) | `fern/cloud-ui/credentials/overview.mdx` | conceptual | Reader understands how Skyvern's credential storage keeps secrets out of LLMs and which types are available in the UI. |
| 46 | Password Credentials (UI) | `fern/cloud-ui/credentials/passwords.mdx` | how-to | Reader can add, edit, and delete a password credential and attach it to a Login block in a workflow. |
| 47 | Credit Card Credentials (UI) | `fern/cloud-ui/credentials/credit-cards.mdx` | how-to | Reader can add a credit card credential and use it in a workflow that completes a purchase. |
| 48 | 2FA / TOTP Setup (UI) | `fern/cloud-ui/credentials/totp.mdx` | how-to | Reader can add a TOTP secret in the UI so Skyvern generates 2FA codes automatically during login tasks. |

### Account & Settings (UI)

| # | Page | Proposed Path | Type | Outcome |
|---|------|--------------|------|---------|
| 49 | API Keys | `fern/cloud-ui/settings/api-keys.mdx` | how-to | Reader can locate, create, and revoke API keys in Skyvern Cloud settings for use with the SDK or REST API. |
| 50 | Billing & Usage | `fern/cloud-ui/settings/billing.mdx` | how-to | Reader can view their credit balance, understand per-step charges, access invoices, and upgrade their plan. |
| 51 | Organization Settings | `fern/cloud-ui/settings/organization.mdx` | how-to | Reader can update organization details and manage team members and roles in their Skyvern Cloud account. |
| 52 | Profile Settings | `fern/cloud-ui/settings/profile.mdx` | how-to | Reader can update their email, change their password, manage notifications, and delete their account. |

### Tips & Troubleshooting (UI)

| # | Page | Proposed Path | Type | Outcome |
|---|------|--------------|------|---------|
| 53 | Common UI Issues | `fern/cloud-ui/troubleshooting/common-issues.mdx` | how-to | Reader can diagnose and resolve the most frequent UI problems such as workflows not saving, missing recordings, or credential selection failures. |

---

## Tab 3: SDK Reference

All pages below are **new** unless noted.

### Getting Started (SDK)

| # | Page | Proposed Path | Type | Outcome |
|---|------|--------------|------|---------|
| 54 | Installation | `fern/sdk/getting-started/installation.mdx` | how-to | Reader can install the Skyvern Python SDK and satisfy all prerequisites on any supported platform. |
| 55 | Quick Start | `fern/sdk/getting-started/quickstart.mdx` | tutorial | Reader can authenticate and run their first task with the SDK in under five minutes using async/await. |
| 56 | Configuration | `fern/sdk/getting-started/configuration.mdx` | reference | Reader can configure the SDK client for Skyvern Cloud, a self-hosted instance, or local embedded mode. |

### Client Reference

| # | Page | Proposed Path | Type | Outcome |
|---|------|--------------|------|---------|
| 57 | Skyvern Client | `fern/sdk/client/overview.mdx` | reference | Reader understands the Skyvern client constructor options and how to use it as an async context manager. |
| 58 | Running Tasks | `fern/sdk/client/tasks.mdx` | reference | Reader can call run_task(), login(), and download_files() with the correct parameters and handle their return types. |
| 59 | Managing Runs | `fern/sdk/client/runs.mdx` | reference | Reader can poll run status, cancel runs, retrieve step timelines, and replay failed webhooks via the SDK. |
| 60 | Workflows | `fern/sdk/client/workflows.mdx` | reference | Reader can create, run, update, and delete workflows programmatically using the SDK. |
| 61 | Browser Sessions | `fern/sdk/client/browser-sessions.mdx` | reference | Reader can create, inspect, and close persistent browser sessions via the SDK. |
| 62 | Browser Profiles | `fern/sdk/client/browser-profiles.mdx` | reference | Reader can create browser profiles from completed runs and attach them to future runs via the SDK. |
| 63 | Credentials | `fern/sdk/client/credentials.mdx` | reference | Reader can manage credentials and push TOTP codes to Skyvern programmatically using the SDK. |
| 64 | Artifacts | `fern/sdk/client/artifacts.mdx` | reference | Reader can retrieve and filter artifacts for any run using the SDK. |
| 65 | Files | `fern/sdk/client/files.mdx` | reference | Reader can upload files for use in tasks via the SDK's upload_file() method. |
| 66 | Scripts (Advanced) | `fern/sdk/client/scripts.mdx` | reference | Reader can create, deploy, and execute cached Playwright scripts through the SDK's scripts API. |

### Browser Control

| # | Page | Proposed Path | Type | Outcome |
|---|------|--------------|------|---------|
| 67 | Overview | `fern/sdk/browser-control/overview.mdx` | conceptual | Reader understands when to use direct browser control versus AI-powered tasks and how the two modes interoperate. |
| 68 | Launching Browsers | `fern/sdk/browser-control/launching.mdx` | how-to | Reader can launch a local or cloud browser using the SDK and obtain a SkyvernBrowser instance. |
| 69 | Connecting to Browsers | `fern/sdk/browser-control/connecting.mdx` | how-to | Reader can attach the SDK to an existing browser via a CDP URL or an existing cloud browser session. |
| 70 | SkyvernBrowser Reference | `fern/sdk/browser-control/skyvern-browser.mdx` | reference | Reader can use all SkyvernBrowser methods to manage pages, cookies, storage, network routes, and permissions. |

### Types Reference

| # | Page | Proposed Path | Type | Outcome |
|---|------|--------------|------|---------|
| 71 | Run Types | `fern/sdk/types/runs.mdx` | reference | Reader can correctly type-hint run requests and responses and interpret every RunStatus and RunEngine value. |
| 72 | Workflow Types | `fern/sdk/types/workflows.mdx` | reference | Reader can construct valid workflow definitions and interpret workflow run timeline responses. |
| 73 | Block Types | `fern/sdk/types/blocks.mdx` | reference | Reader can find the correct class and required fields for any of the 25 block types when building workflows in code. |
| 74 | Browser Types | `fern/sdk/types/browser.mdx` | reference | Reader can work with BrowserSessionResponse, BrowserProfile, and SkyvernBrowser objects returned by the SDK. |
| 75 | Credential Types | `fern/sdk/types/credentials.mdx` | reference | Reader can construct the correct credential payload for each credential type and interpret credential response objects. |
| 76 | Artifact Types | `fern/sdk/types/artifacts.mdx` | reference | Reader can filter and interpret artifacts by ArtifactType when retrieving run outputs. |
| 77 | Proxy Types | `fern/sdk/types/proxy.mdx` | reference | Reader can choose the correct ProxyLocation enum value or construct a GeoTarget object for any geographic target. |

---

## Summary Counts

| Status | Count |
|--------|-------|
| `exists` (already in fern/) | 31 |
| `new` (needs to be created) | 46 |
| **Total** | **77** |

### Priority New Pages (per DOC-PLAN developer notes)
1. CAPTCHA handling page (mentioned as high priority in DOC-PLAN)
2. File operations FAQ (common Pylon support question)
3. Auth troubleshooting guide (many Pylon tickets)
4. SDK Quick Start + Client reference (Phase 1–5 progression)
5. Cloud UI Overview + Discover page (Tab 2 entry points)
