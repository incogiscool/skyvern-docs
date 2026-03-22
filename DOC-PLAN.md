## SKYVERN DOCS \- CURRENT STRUCTURE

\================================

Getting Started  
├── Introduction  
├── Quickstart  
├── Skyvern In Action  
└── Prompting and Troubleshooting Guide

Tasks  
├── Run Tasks  
├── Visualizing Results  
├── Cancel Task Runs  
├── Webhooks FAQ  
├── Proxy & Geo Targeting

Workflows  
├── Manage Workflows  
├── Workflow Blocks  
├── Workflow Parameters  
├── Run Workflows  
├── Consistency and Reliability

Credentials  
├── Overview  
├── Password Management  
├── Credit Card Management  
├── 2FA Support (TOTP)  
├── Bitwarden  
└── Custom Credential Service

Browser Sessions (Beta)  
├── Introduction  
└── Browser Profiles

Observability  
└── Overview

Integrations  
├── MCP server  
├── CLI and Skills  
├── Zapier  
├── Make  
├── N8N  
├── Workato  
└── Ollama \+ LiteLLM

## Product

┌─────────────────────────────────────────────────────────────────────────────┐  
│                           SKYVERN PRODUCTS                                  │  
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐  
│                                                                             │  
│  1\. SKYVERN CLOUD UI                                                        │  
│  ═══════════════════                                                        │  
│                                                                             │  
│  WHO: Non-technical users, Automation Operators                             │  
│  WHAT: Web dashboard at app.skyvern.com                                     │  
│  HOW: Point-and-click workflow builder, no code                             │  
│                                                                             │  
│  ┌─────────────────────────────────────────────────────────────────────┐    │  
│  │  • Discover tab (templates)                                         │    │  
│  │  • Visual workflow editor                                           │    │  
│  │  • Run history & recordings                                         │    │  
│  │  • Credential management                                            │    │  
│  │  • Settings & API keys                                              │    │  
│  └─────────────────────────────────────────────────────────────────────┘    │  
│                                                                             │  
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐  
│                                                                             │  
│  2\. API/SDK                                                                 │  
│  ═════════                                                                  │  
│                                                                             │  
│  WHO: Developers, Integration Engineers                                     │  
│  WHAT: Python SDK \+ REST API                                                │  
│                                                                             │  
│  TWO EXECUTION MODES:                                                       │  
│                                                                             │  
│  ┌─────────────────────────────┐    ┌─────────────────────────────┐        │  
│  │  A) CLOUD BROWSER           │    │  B) LOCAL BROWSER           │        │  
│  │     (Default)               │    │     (Hybrid)                │        │  
│  ├─────────────────────────────┤    ├─────────────────────────────┤        │  
│  │                             │    │                             │        │  
│  │  Browser runs on            │    │  Browser pops up on         │        │  
│  │  Skyvern's servers          │    │  YOUR machine               │        │  
│  │                             │    │                             │        │  
│  │  • Zero setup               │    │  • See it happen live       │        │  
│  │  • Built-in proxies         │    │  • Access local network     │        │  
│  │  • Pay per step             │    │  • Debug visually           │        │  
│  │  • Fully managed            │    │  • Still uses Cloud API     │        │  
│  │                             │    │                             │        │  
│  │  client.run\_task(...)       │    │                             │        │  
│  │                             │    │                             │        │  
│  └─────────────────────────────┘    └─────────────────────────────┘        │  
│                                                                            │  
└────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐  
│                                                                             │  
│  3\. SELF-HOSTED / ENTERPRISE                                                │  
│  ═══════════════════════════                                                │  
│                                                                             │  
│  WHO: Enterprise, Regulated Industries, High-Volume Users                   │  
│  WHAT: Run EVERYTHING on your infrastructure                                │  
│                                                                             │  
│  ┌─────────────────────────────────────────────────────────────────────┐    │  
│  │                                                                     │    │  
│  │  YOUR INFRASTRUCTURE                                                │    │  
│  │  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐            │    │  
│  │  │   Skyvern    │  │   Browser     │  │   Your LLM    │             │    │   
│  │  │   Server     │──│   (Docker)    │  │   API Keys    │             │    │  
│  │  │   (Docker)   │  │               │  │               │             │    │  
│  │  └───────────────┘  └───────────────┘  └───────────────┘            │    │  
│  │                                                                     │    │  
│  │  • Full data privacy (nothing leaves your network)                  │    │  
│  │  • Bring your own LLM keys (OpenAI, Anthropic, etc.)                │    │  
│  │  • Custom scaling                                                   │    │  
│  │  • Docker Compose / Kubernetes                                      │    │  
│  │  • Requires setup & maintenance                                     │    │  
│  │                                                                     │    │    
│  └─────────────────────────────────────────────────────────────────────┘    │    
│                                                                             │  
└─────────────────────────────────────────────────────────────────────────────┘

## 

## 

## 

## Customer Personas

| Persona | Who They Are | Technical Level | Primary Interface | Key Features Used |
| :---- | :---- | :---- | :---- | :---- |
| **Integration Developer** | Backend engineers, automation engineers, DevOps | High \- comfortable with APIs, async code, webhooks | REST API, Python SDK | Tasks, Workflows (via API), Browser Sessions, Browser Profiles, Webhooks |
| **Automation Operator** | BizOps, RevOps, Operations analysts | Medium \- comfortable with no-code tools, some technical knowledge | Cloud UI: Discover page, Visual Editor, Run History | Tasks (via UI), Workflows (via Editor), Credentials, Zapier/Make/n8n integrations |
| **Decision Maker** | C-suite, Engineering Managers, IT Leads, Security Officers | Varies \- evaluating for team adoption | Documentation, Demo calls, Security reviews | N/A (evaluating capabilities) |

## API/SDK Features

╔══════════════════════════════════════════════════════════════════════════════╗  
║                    SKYVERN API/SDK \- DEVELOPER PROGRESSION                   ║  
╚══════════════════════════════════════════════════════════════════════════════╝

┌──────────────────────────────────────────────────────────────────────────────┐  
│  PHASE 1: FIRST SUCCESS                                                      │  
│  ─────────────────────────────────────────────────────────────────────────── │  
│  What they know: "I want to automate something in a browser"                 │  
│                                                                              │  
│  METHODS:                                                                    │  
│    • run\_task(prompt, url)      → Start automation                           │  
│    • get\_run(task\_id)           → Check status & results                     │  
│    • get\_run\_artifacts()        → Watch the recording  ★ AHA MOMENT          │  
│                                                                              │  
│  OUTCOME: "It works\!"                                                        │  
└──────────────────────────────────────────────────────────────────────────────┘  
                                       │  
                                       ▼  
┌──────────────────────────────────────────────────────────────────────────────┐  
│  PHASE 2: CONTROL OUTPUT                                                     │  
│  ─────────────────────────────────────────────────────────────────────────── │  
│  What they know: "It works but I need structured data"                       │  
│                                                                              │  
│  NEW PARAMS:                                                                 │  
│    • data\_extraction\_schema     → JSON Schema for typed output               │  
│    • error\_code\_mapping         → Custom error handling                      │  
│    • max\_steps                  → Cost control                               │  
│                                                                              │  
│  OUTCOME: "I get the data I need"                                            │  
└──────────────────────────────────────────────────────────────────────────────┘  
                                       │  
                                       ▼  
┌──────────────────────────────────────────────────────────────────────────────┐  
│  PHASE 3: HANDLE AUTH                                                        │  
│  ─────────────────────────────────────────────────────────────────────────── │  
│  What they know: "Real sites need login"                                     │  
│                                                                              │  
│  METHODS:                                                                    │  
│    • create\_credential()        → Store username/password securely           │  
│    • send\_totp\_code()           → Handle 2FA (authenticator/email/SMS)       │  
│                                                                              │  
│  CREDENTIAL TYPES: password | credit\_card | secret                           │  
│  2FA TYPES: authenticator | email | text                                     │  
│                                                                              │  
│  OUTCOME: "It handles real sites"                                            │  
└──────────────────────────────────────────────────────────────────────────────┘  
                                       │  
                                       ▼  
┌──────────────────────────────────────────────────────────────────────────────┐  
│  PHASE 4: WORKFLOWS                                                          │  
│  ─────────────────────────────────────────────────────────────────────────── │  
│  What they know: "Single task isn't enough"                                  │  
│                                                                              │  
│  METHODS:                                                                    │  
│    • create\_workflow()          → Define multi-step automation               │  
│    • run\_workflow()             → Execute with parameters                    │  
│                                                                              │  
│  21 BLOCK TYPES:                                                             │  
│    Navigation:   NavigationBlock, UrlBlock                                   │  
│    Actions:      ActionBlock, LoginBlock                                     │  
│    Extraction:   ExtractionBlock, TaskBlock, TaskV2Block                     │  
│    Control:      ForLoopBlock, ConditionalBlock, WaitBlock                   │  
│    Files:        FileDownloadBlock, FileUploadBlock, PdfParserBlock          │  
│    Integration:  HttpRequestBlock, SendEmailBlock, S3 blocks                 │  
│    Other:        CodeBlock, TextPromptBlock, ValidationBlock                 │  
│                                                                              │  
│  OUTCOME: "I can chain complex flows"                                        │  
└──────────────────────────────────────────────────────────────────────────────┘  
                                       │  
                                       ▼  
┌──────────────────────────────────────────────────────────────────────────────┐  
│  PHASE 5: OPTIMIZE                                                           │  
│  ─────────────────────────────────────────────────────────────────────────── │  
│  What they know: "Login every time is slow/costly"                           │  
│                                                                              │  
│  BROWSER SESSIONS (temporary):                                               │  
│    • create\_browser\_session()   → Live browser stays open                    │  
│    • run\_task(browser\_session\_id=...) → Reuse same browser                   │  
│    • close\_browser\_session()    → Done for the day                           │  
│                                                                              │  
│  BROWSER PROFILES (persistent):                                              │  
│    • create\_browser\_profile()   → Save logged-in state forever               │  
│    • run\_workflow(browser\_profile\_id=...) → Skip login on future runs        │  
│                                                                              │  
│  OUTCOME: "It's fast and cost-effective"                                     │  
└──────────────────────────────────────────────────────────────────────────────┘  
                                       │  
                                       ▼  
┌──────────────────────────────────────────────────────────────────────────────┐  
│  PHASE 6: PRODUCTION                                                         │  
│  ─────────────────────────────────────────────────────────────────────────── │  
│  What they know: "I need this running reliably at scale"                     │  
│                                                                              │  
│  NEW PARAMS:                                                                 │  
│    • webhook\_url                → Don't poll, get notified                   │  
│    • proxy\_location             → Route through specific geography           │  
│                                                                              │  
│  25+ PROXY LOCATIONS:                                                        │  
│    US States: CA, NY, TX, FL, WA                                             │  
│    Countries: GB, DE, FR, JP, AU, IN, ES, IE, BR, and more                   │  
│                                                                              │  
│  OUTCOME: "It runs at scale"                                                 │  
└──────────────────────────────────────────────────────────────────────────────┘

## New Proposed Structure

┌────────────────┬────────────────┬────────────────┬────────────────┬────────────────┐  
│  DOCUMENTATION │ SKYVERN CLOUD  │   SDK          │ API REFERENCE  │   RESOURCES    │  
├────────────────┼────────────────┼────────────────┼────────────────┼────────────────┤  
│                │                │                │                │                │  
│ Guides for     │ How to use     │ Python client  │ REST endpoints │ Cookbooks,     │  
│ developers     │ the web UI     │ methods,       │ auto-generated │ prompting      │  
│ using the      │ to build       │ types, and     │ from OpenAPI   │ guide,         │  
│ API/SDK        │ automations    │ examples       │ spec           │ troubleshooting│  
│                │                │                │                │                │  
└────────────────┴────────────────┴────────────────┴────────────────┴────────────────┘

### 

### Tab 1: Documentation

Getting Started ✅  
│  
├── Introduction  
│   \- What Skyvern is in one paragraph  
│   \- How it works (LLM \+ vision \+ browser)  
│   \- Three ways to use it: Cloud UI, API/SDK, Self-Hosted  
│   \- "This tab is for developers using the API/SDK"  
│  
├── Quickstart  
│   \- Get API key  
│   \- Install SDK  
│   \- Run task (Cloud browser by default)  
│   \- See results and watch recording  
│   \- Optional: Run with local browser instead (one code snippet showing the        difference)  
│  
└── Core Concepts  
    \- Tasks: single automation jobs  
    \- Workflows: multi-step automations using blocks  
    \- Blocks: building units of workflows (21 types)  
    \- Runs: execution instances with lifecycle (created → running → completed)  
    \- Credentials: secure storage for auth (never exposed to LLM)  
    \- Browser Sessions: temporary live browser state  
    \- Browser Profiles: persistent saved browser state  
    \- Artifacts: outputs like recordings, screenshots, logs

Running Automations ✅  
│  
├── Run a Task  
│   \- What a task is  
│   \- Basic run\_task() with prompt and URL  
│   \- Understanding the response  
│   \- Checking status with get\_run()  
│  
├── Task Parameters  
│   \- All parameters: engine, max\_steps, proxy\_location, webhook\_url, etc.  
│   \- AI engines: skyvern-2.0, skyvern-1.0, openai-cua, anthropic-cua  
│   \- Code examples for common configurations  
│  
└── Extract Structured Data  
    \- Defining data\_extraction\_schema  
    \- JSON Schema format  
    \- Examples: single values, lists, nested objects  
    \- How extracted data appears in response

Handling Authentication ✅  
│  
├── Store Credentials  
│   \- Why use Skyvern's credential storage  
│   \- Password credentials  
│   \- Credit card credentials  
│   \- Secret credentials (API keys, tokens)  
│  
└── Handle 2FA  
    \- Method 1: Store TOTP secret (Skyvern generates codes)  
    \- Method 2: Skyvern pulls from your endpoint  
    \- Method 3: Push codes to Skyvern  
    \- Method 4: One-time login links  
    \- Password manager integrations (Bitwarden, 1Password)  
|  
├── Troubleshooting Login Failures (NEW)  
│   \- Common failure patterns  
│   \- Portal-specific issues  
│   \- Multi-step login timeouts  
│   \- When credentials don't get entered

Specific guides for bitwarden as well.

Multi-Step Automations ✅  
│  
├── Introduction to Workflows  
│   \- What workflows are and why you need them  
│   \- Tasks vs Workflows: when to use each  
│   \- Workflow structure: blocks in sequence  
│  
├── Build a Workflow  
│   \- Creating via API (JSON/YAML)  
│   \- Adding blocks  
│   \- Connecting blocks with parameters  
│   \- Running and testing  
│  
├── Workflow Blocks Reference  
│   \- All 21 blocks organized by purpose  
│   \- Parameters for each block  
│   \- Code examples  
|  
├── File Operations (NEW)  
│   \- Downloading files (FileDownloadBlock)  
│   \- Parsing CSV/Excel/PDF (FileParserBlock)  
│   \- Uploading to S3 (FileUploadBlock)  
│   \- Passing files between blocks  
│  
└── Workflow Parameters  
    \- Input parameters (pass data at runtime)  
    \- Output parameters (get data between blocks)  
    \- Jinja templating syntax

Going to Production ✅  
│  
├── Webhooks  
│   \- Why webhooks (don't poll, get notified)  
│   \- Setting webhook\_url  
│   \- Payload structure  
│   \- Retry behavior  
│  
├── Proxy & Geolocation  
│   \- Available locations (25+)  
│   \- When to use geo-targeting  
│   \- Setting proxy\_location  
│  
├── Error Handling  
│   \- Common failure modes  
│   \- Using error\_code\_mapping  
│   \- Interpreting failure reasons  
│  
└── Reliability Tips  
    \- Writing robust prompts  
    \- Handling dynamic pages  
    \- Validation strategies  
    \- Keyboard Actions (Pylon)

├── CAPTCHA & Bot Detection (NEW)

Debugging (in-review)  
│  
├── Using Artifacts   
│   \- Available artifacts: recordings, screenshots, logs, HAR  
│   \- Retrieving with get\_run\_artifacts()  
│   \- What each artifact tells you  
│  
└── Troubleshooting Guide  
    \- Common issues and solutions  
    \- Reading the step timeline  
    \- When to adjust prompts vs parameters

Optimization   
│  
├── Browser Sessions  
│   \- What sessions are (live browser between calls)  
│   \- When to use sessions  
│   \- Creating, using, closing sessions  
│  
├── Browser Profiles  
│   \- What profiles are (saved state for reuse)  
│   \- Creating from workflow runs  
│   \- Skipping login on future runs  
│  
└── Cost Control  
    \- How billing works (per step)  
    \- Using max\_steps  
    \- Efficient prompt writing

Self-Hosted Deployment (in-review)  
│  
├── Overview  
│   \- When to self-host (data privacy, compliance, high volume)  
│   \- Architecture: your server \+ your browser \+ your LLM keys  
│   \- What you gain vs what you lose (no managed proxies, you handle scaling)  
│  
├── Docker Setup  
│   \- Prerequisites  
│   \- Docker Compose configuration  
│   \- Required services and containers  
│   \- Environment variables reference  
│  
├── LLM Configuration  
│   \- Connecting OpenAI  
│   \- Connecting Anthropic  
│   \- Using other providers (Azure, local models)  
│   \- Model selection and cost implications  
│  
├── Browser Configuration  
│   \- Browser container setup  
│   \- Headless vs headed mode  
│   \- Scaling browser instances  
│  
└── Proxy Setup  
    \- Why you need your own proxies (self-hosted has none built-in)  
    \- Configuring proxy providers  
    \- Residential vs datacenter proxies

├── Enterprise (NEW)  
│   \- SSO/SAML setup  
│   \- Organizations & Teams  
│   \- Audit logs

Integrations (in-review)  
│  
├── MCP Server  
│   \- Using Skyvern with AI applications via Model Context Protocol  
│  
├── Zapier  
│   \- Connecting Skyvern to Zapier workflows  
│  
├── Make  
│   \- Make.com integration  
│  
└── Other Integrations  
    \- **N8N**, Workato, Ollama \+ LiteLLM

Notes from Pylon tickets:

- CAPTCHA seems like a big issue, how do we tackle this? I’ll add a page accordingly  
- File operations seem to be a common question   
- Keyboard presses are not yet supported but we can document some common scenarios and workarounds to them. (higher priority is building this out)  
- Lot of auth troubleshooting in pylon, adding a section for that here.

### Tab 2: Skyvern Cloud (UI)

Skyvern Cloud (UI) │  
├── Getting Started (in-review)  
│   │  
│   ├── UI Overview  
│   │   \- What Skyvern Cloud is (app.skyvern.com)  
│   │   \- Main navigation: Discover, Workflows, History, Credentials, Settings  
│   │   \- When to use UI vs API/SDK  
│   │   \- Quick tour of the interface  
│   │  
│   └── Your First Task  
│       \- Navigating to Discover page  
│       \- Entering a URL and prompt  
│       \- Choosing an engine  
│       \- Running the task  
│       \- Watching the live view and recording  
│  
│  
├── Running Tasks (in review)  
│   │  
│   ├── The Discover Page  
│   │   \- What Discover is for (ad-hoc task execution)  
│   │   \- The task creation form  
│   │   \- Available options: URL, prompt, engine, max steps  
│   │   \- Advanced settings: proxy location, webhook URL  
│   │   \- Data extraction schema (JSON input)  
│   │  
│   └── Watching Live Execution  
│       \- Live browser view  
│       \- When live view is available  
│       \- Stopping a running task  
│  
│  
├── Building Workflows (in review)  
│   │  
│   ├── Workflows List  
│   │   \- Viewing all workflows  
│   │   \- Creating a new workflow  
│   │   \- Cloning a workflow (for versioning)  
│   │   \- Deleting a workflow  
│   │   \- Export/Import (YAML/JSON)  
│   │  
│   ├── The Workflow Editor  
│   │   \- Editor layout and navigation  
│   │   \- Adding blocks (+ button)  
│   │   \- Configuring blocks (side panel)  
│   │   \- Connecting blocks in sequence  
│   │   \- Deleting blocks (... menu)  
│   │   \- Saving your workflow (not auto-saved\!)  
│   │  
│   ├── Working with Parameters  
│   │   \- What parameters are  
│   │   \- Creating workflow parameters  
│   │   \- Using parameters in blocks (Jinja syntax)  
│   │   \- Credential parameters  
│   │   \- Block output parameters  
│   │   \- Reserved parameters (current\_value)  
│   │  
│   ├── Block Configuration  
│   │   \- Navigation blocks (URL, NavigationV2)  
│   │   \- Task blocks (main automation)  
│   │   \- Action blocks  
│   │   \- Extraction blocks  
│   │   \- Login blocks (with credential selection)  
│   │   \- Control flow (For Loop, Conditional)  
│   │   \- File blocks (Download, Parse, Upload)  
│   │   \- Code blocks  
│   │   \- Other blocks (Wait, SendEmail, HttpRequest)  
│   │  
│   └── Running Workflows  
│       \- The Run button  
│       \- Parameter input modal  
│       \- Selecting credentials at runtime  
│       \- Viewing queued/running status  
│  
│  
├── Viewing Results  
│   │  
│   ├── Run History  
│   │   \- Navigating to History page  
│   │   \- Filtering runs (by status, date, type)  
│   │   \- Understanding run statuses (created, running, completed, failed)  
│   │   \- Clicking into a run  
│   │  
│   ├── Run Details  
│   │   \- Actions tab (step-by-step with screenshots)  
│   │   \- Recording tab (full video playback)  
│   │   \- Parameters tab (inputs you provided)  
│   │   \- Output tab (extracted data, results)  
│   │  
│   ├── Diagnostic Logs  
│   │   \- What diagnostics contain  
│   │   \- Annotated screenshots  
│   │   \- Element tree  
│   │   \- Prompts sent to LLM  
│   │   \- Raw LLM requests/responses  
│   │   \- Using diagnostics for debugging  
│   │  
│   └── Downloading Artifacts  
│       \- Available artifacts (recording, screenshots, HAR)  
│       \- How to download  
│  
│  
├── Managing Credentials  
│   │  
│   ├── Credentials Overview  
│   │   \- Why use Skyvern's credential storage  
│   │   \- Security: credentials never sent to LLMs  
│   │   \- Credential types available  
│   │  
│   ├── Password Credentials  
│   │   \- Adding a password credential  
│   │   \- Editing and deleting  
│   │   \- Using in workflows (Login block)  
│   │  
│   ├── Credit Card Credentials  
│   │   \- Adding a credit card  
│   │   \- Supported card types (Visa, Mastercard)  
│   │   \- Using in purchase workflows  
│   │  
│   └── 2FA / TOTP Setup  
│       \- Adding TOTP secrets  
│       \- How Skyvern generates codes  
│       \- Viewing active TOTP credentials  
│  
│  
├── Account & Settings  
│   │  
│   ├── API Keys  
│   │   \- Where to find your API key  
│   │   \- Creating new API keys  
│   │   \- Revoking keys  
│   │   \- Using keys with SDK/API  
│   │  
│   ├── Billing & Usage  
│   │   \- Viewing your balance/credits  
│   │   \- Understanding usage (per-step billing)  
│   │   \- Accessing invoices  
│   │   \- Upgrading your plan  
│   │  
│   ├── Organization Settings  
│   │   \- Organization name and details  
│   │   \- Team members (if applicable)  
│   │   \- Roles and permissions (Enterprise)  
│   │  
│   └── Profile Settings  
│       \- Email and notifications  
│       \- Password changes  
│       \- Account deletion (GDPR)  
│  
│  
└── Tips & Troubleshooting  
    │  
    ├── Common UI Issues  
    │   \- "Save" button not working  
    │   \- Workflow not running (check parameters)  
    │   \- Missing recordings  
    │   \- Can't select credentials in blocks  
    │  
    └── Keyboard Shortcuts (if any)  
        \- Editor shortcuts  
        \- Navigation shortcuts

### Tab 3: SDK Reference \- TS \+ SDK 

SDK  
│  
├── Getting Started  
│   │  
│   ├── Installation  
│   │   \- pip install skyvern  
│   │   \- Requirements (Python 3.9+)  
│   │   \- Optional dependencies  
│   │  
│   ├── Quick Start  
│   │   \- Import and initialize client  
│   │   \- Authentication with API key  
│   │   \- Your first run\_task() call  
│   │   \- Async/await pattern  
│   │  
│   └── Configuration  
│       \- API key setup  
│       \- Base URL (cloud vs self-hosted)  
│       \- Request options (timeout, retries)  
│       \- Local mode: Skyvern.local()  
│  
│  
├── Client Reference  
│   │  
│   ├── Skyvern (Main Client)  
│   │   \- Constructor: Skyvern(api\_key, base\_url)  
│   │   \- Skyvern.local() for self-hosted  
│   │   \- Async context manager usage  
│   │   \- close() / aclose()  
│   │  
│   ├── Running Tasks  
│   │   │  
│   │   ├── run\_task()  
│   │   │   \- All parameters with descriptions  
│   │   │   \- Return type: TaskRunResponse  
│   │   │   \- wait\_for\_completion option  
│   │   │   \- Code examples  
│   │   │  
│   │   ├── login()  
│   │   │   \- Credential types (password, bitwarden, etc.)  
│   │   │   \- Parameters  
│   │   │   \- Return type: WorkflowRunResponse  
│   │   │  
│   │   └── download\_files()  
│   │       \- Navigation goal for downloads  
│   │       \- Parameters  
│   │       \- Return type: WorkflowRunResponse  
│   │  
│   ├── Managing Runs  
│   │   │  
│   │   ├── get\_run()  
│   │   │   \- Checking run status  
│   │   │   \- Return type: GetRunResponse  
│   │   │  
│   │   ├── cancel\_run()  
│   │   │   \- Stopping a running task  
│   │   │  
│   │   ├── get\_run\_timeline()  
│   │   │   \- Step-by-step execution history  
│   │   │   \- Return type: List\[WorkflowRunTimeline\]  
│   │   │  
│   │   └── retry\_run\_webhook()  
│   │       \- Retrying failed webhook delivery  
│   │  
│   ├── Workflows  
│   │   │  
│   │   ├── run\_workflow()  
│   │   │   \- workflow\_id, parameters  
│   │   │   \- wait\_for\_completion option  
│   │   │   \- Return type: WorkflowRunResponse  
│   │   │  
│   │   ├── get\_workflows()  
│   │   │   \- Listing workflows (pagination)  
│   │   │   \- Filtering options  
│   │   │  
│   │   ├── create\_workflow()  
│   │   │   \- JSON or YAML definition  
│   │   │   \- WorkflowCreateYamlRequest  
│   │   │  
│   │   ├── update\_workflow()  
│   │   │   \- Modifying existing workflow  
│   │   │  
│   │   └── delete\_workflow()  
│   │       \- Removing a workflow  
│   │  
│   ├── Browser Sessions  
│   │   │  
│   │   ├── create\_browser\_session()  
│   │   │   \- timeout, proxy\_location  
│   │   │   \- Return type: BrowserSessionResponse  
│   │   │  
│   │   ├── get\_browser\_session()  
│   │   │   \- Getting session details  
│   │   │  
│   │   ├── get\_browser\_sessions()  
│   │   │   \- Listing all sessions  
│   │   │  
│   │   └── close\_browser\_session()  
│   │       \- Ending a session  
│   │  
│   ├── Browser Profiles  
│   │   │  
│   │   ├── create\_browser\_profile()  
│   │   │   \- From session or workflow run  
│   │   │   \- name, description  
│   │   │  
│   │   ├── get\_browser\_profile()  
│   │   │   \- Getting profile details  
│   │   │  
│   │   ├── list\_browser\_profiles()  
│   │   │   \- Listing all profiles  
│   │   │  
│   │   └── delete\_browser\_profile()  
│   │       \- Removing a profile  
│   │  
│   ├── Credentials  
│   │   │  
│   │   ├── create\_credential()  
│   │   │   \- credential\_type, credential data  
│   │   │   \- Supported types  
│   │   │  
│   │   ├── get\_credential()  
│   │   │   \- Retrieving credential metadata  
│   │   │  
│   │   ├── get\_credentials()  
│   │   │   \- Listing all credentials (paginated)  
│   │   │  
│   │   ├── delete\_credential()  
│   │   │   \- Removing a credential  
│   │   │  
│   │   └── send\_totp\_code()  
│   │       \- Pushing 2FA codes to Skyvern  
│   │  
│   ├── Artifacts  
│   │   │  
│   │   ├── get\_artifact()  
│   │   │   \- Getting a specific artifact  
│   │   │   \- Return type: Artifact  
│   │   │  
│   │   └── get\_run\_artifacts()  
│   │       \- All artifacts for a run  
│   │       \- Filtering by artifact\_type  
│   │  
│   ├── Files  
│   │   │  
│   │   └── upload\_file()  
│   │       \- Uploading files for use in tasks  
│   │       \- Return type: UploadFileResponse  
│   │  
│   └── Scripts (Advanced)  
│       │  
│       ├── create\_script()  
│       ├── get\_script()  
│       ├── get\_scripts()  
│       ├── deploy\_script()  
│       └── run\_sdk\_action()  
│           \- Low-level action execution  
│  
│  
├── Browser Control  
│   │  
│   ├── Overview  
│   │   \- When to use direct browser control  
│   │   \- Cloud vs local browser  
│   │   \- SkyvernBrowser vs BrowserSession  
│   │  
│   ├── Launching Browsers  
│   │   │  
│   │   ├── launch\_local\_browser()  
│   │   │   \- headless, port, args, user\_data\_dir  
│   │   │   \- Return type: SkyvernBrowser  
│   │   │  
│   │   ├── launch\_cloud\_browser()  
│   │   │   \- timeout, proxy\_location  
│   │   │   \- Return type: SkyvernBrowser  
│   │   │  
│   │   └── use\_cloud\_browser()  
│   │       \- Context manager version  
│   │  
│   ├── Connecting to Browsers  
│   │   │  
│   │   ├── connect\_to\_browser\_over\_cdp()  
│   │   │   \- Using existing browser via CDP URL  
│   │   │  
│   │   └── connect\_to\_cloud\_browser\_session()  
│   │       \- Connecting to existing cloud session  
│   │  
│   └── SkyvernBrowser Reference  
│       │  
│       ├── Page Management  
│       │   \- new\_page()  
│       │   \- get\_working\_page()  
│       │   \- close()  
│       │  
│       ├── Cookie Management  
│       │   \- cookies()  
│       │   \- add\_cookies()  
│       │   \- clear\_cookies()  
│       │  
│       ├── State & Storage  
│       │   \- storage\_state()  
│       │   \- set\_geolocation()  
│       │   \- set\_offline()  
│       │  
│       ├── Network  
│       │   \- set\_extra\_http\_headers()  
│       │   \- route() / unroute()  
│       │   \- route\_from\_har()  
│       │  
│       ├── Permissions  
│       │   \- grant\_permissions()  
│       │   \- clear\_permissions()  
│       │  
│       └── Events  
│           \- on() / once()  
│           \- expect\_page()  
│           \- expect\_console\_message()  
│           \- wait\_for\_event()  
│  
│  
├── Types Reference  
│   │  
│   ├── Run Types  
│   │   \- TaskRunResponse  
│   │   \- WorkflowRunResponse  
│   │   \- GetRunResponse  
│   │   \- RunStatus (enum)  
│   │   \- RunEngine (enum)  
│   │  
│   ├── Workflow Types  
│   │   \- Workflow  
│   │   \- WorkflowDefinition  
│   │   \- WorkflowCreateYamlRequest  
│   │   \- WorkflowParameter  
│   │   \- WorkflowRunTimeline  
│   │  
│   ├── Block Types  
│   │   \- TaskBlock, TaskV2Block  
│   │   \- NavigationBlock, UrlBlock  
│   │   \- ActionBlock, ExtractionBlock  
│   │   \- LoginBlock, ValidationBlock  
│   │   \- ForLoopBlock, ConditionalBlock  
│   │   \- CodeBlock, TextPromptBlock  
│   │   \- FileDownloadBlock, FileParserBlock  
│   │   \- SendEmailBlock, HttpRequestBlock  
│   │   \- WaitBlock, HumanInteractionBlock  
│   │  
│   ├── Browser Types  
│   │   \- BrowserSessionResponse  
│   │   \- BrowserProfile  
│   │   \- SkyvernBrowser  
│   │   \- SkyvernBrowserPage  
│   │  
│   ├── Credential Types  
│   │   \- CredentialResponse  
│   │   \- PasswordCredentialResponse  
│   │   \- CreditCardCredentialResponse  
│   │   \- SecretCredentialResponse  
│   │   \- CredentialType (enum)  
│   │  
│   ├── Artifact Types  
│   │   \- Artifact  
│   │   \- ArtifactType (enum)  
│   │  
│   ├── Location Types  
│   │   \- ProxyLocation (enum)  
│   │   \- GeoTarget  
│   │  
│   └── Action Types  
│       \- Action  
│       \- ClickAction, InputTextAction  
│       \- SelectOptionAction, UploadFileAction  
│       \- ExtractAction, ValidateAction  
│  
│  
├── Enums Reference  
│   │  
│   ├── RunStatus  
│   │   \- created, queued, running  
│   │   \- completed, failed, terminated  
│   │   \- timed\_out, canceled  
│   │  
│   ├── RunEngine  
│   │   \- skyvern\_v2, skyvern\_v1  
│   │   \- openai\_cua, anthropic\_cua  
│   │   \- ui\_tars  
│   │  
│   ├── ProxyLocation  
│   │   \- RESIDENTIAL, RESIDENTIAL\_ISP  
│   │   \- US states (US-CA, US-NY, etc.)  
│   │   \- Countries (RESIDENTIAL\_GB, etc.)  
│   │   \- NONE  
│   │  
│   ├── ArtifactType  
│   │   \- recording, screenshot  
│   │   \- har\_file, llm\_response  
│   │  
│   ├── BlockType  
│   │   \- All 21 block types  
│   │  
│   └── CredentialType  
│       \- password, credit\_card  
│       \- secret, bitwarden, onepassword  
│  
│  
├── Patterns & Examples  
│   │  
│   ├── Polling for Completion  
│   │   \- Using wait\_for\_completion=True  
│   │   \- Manual polling with get\_run()  
│   │   \- Handling timeouts  
│   │  
│   ├── Error Handling  
│   │   \- Exception types  
│   │   \- Checking RunStatus.failed  
│   │   \- Accessing error details  
│   │  
│   ├── Working with Webhooks  
│   │   \- Setting webhook\_url  
│   │   \- Webhook payload structure  
│   │   \- retry\_run\_webhook() usage  
│   │  
│   ├── Using Browser Sessions  
│   │   \- Creating persistent sessions  
│   │   \- Running multiple tasks in one session  
│   │   \- Creating profiles from sessions  
│   │  
│   └── Local Browser Development  
│       \- launch\_local\_browser() setup  
│       \- Debugging with headed mode  
│       \- Using existing Chrome profile  
│

### Tab 4: Cookbooks

├── Multi-Carrier Insurance Quote Aggregator  
│   \- Collect quotes from 3+ insurance carriers in one workflow  
│   \- ForLoopBlock to iterate over carrier URLs  
│   \- Parameterized form data (driver info, vehicle, coverage)  
│   \- Data extraction schema for quote amounts  
│   \- Error handling for carrier-specific failures  
│   \- ConditionalBlock for different form layouts  
│   │  
│   │  WHY THIS COOKBOOK:  
│   │  • 25% of support tickets from finance/insurance sector  
│   │  • Tickets \#1866, \#2094, \#2114 show carrier portal challenges  
│   │  • Demonstrates multi-site automation pattern  
│   │  • Shows real business value (quote comparison)  
│  
│  
├── State Business Formation Bot  
│   \- File LLC/Corporation with Secretary of State portals  
│   \- LoginBlock with stored credentials  
│   \- Multi-step form filling (entity name, registered agent, etc.)  
│   \- Payment handling with credit card credentials  
│   \- FileDownloadBlock for confirmation documents  
│   \- CAPTCHA retry patterns  
│   \- Webhook notification on completion  
│   │  
│   │  WHY THIS COOKBOOK:  
│   │  • Govassist is major customer; \#2213, \#1963, \#2012 are formation bugs  
│   │  • Government sites have complex, changing layouts (Skyvern's strength)  
│   │  • Shows credential \+ payment \+ file download in one workflow  
│   │  • No competitor covers government form automation  
│  
│  
├── Healthcare Portal Data Extraction  
│   \- Extract data from Cloudflare-protected healthcare portals  
│   \- Browser profiles to persist authenticated state  
│   \- Residential proxies for bot detection avoidance  
│   \- Session reuse across multiple extraction runs  
│   \- Handling 403 errors with smart retries  
│   \- Structured data extraction from tables  
│   │  
│   │  WHY THIS COOKBOOK:  
│   │  • Tickets \#2221, \#2234 specifically ask for TherapyNotes \+ Cloudflare  
│   │  • Healthcare companies are \~15% of customer base  
│   │  • Bot detection is \#1 pain point (15+ CAPTCHA tickets)  
│   │  • Shows browser profiles \+ proxies together (advanced pattern)  
│  
│  
├── Bulk Invoice Downloader with Email Summary  
│   \- Download invoices from vendor portal  
│   \- Parse invoice data from PDFs  
│   \- Loop through multiple invoices  
│   \- Send email summary with totals  
│   \- Pass file data between workflow blocks  
│   │  
│   │  WHY THIS COOKBOOK:  
│   │  • Invoice downloading is existing demo but lacks depth  
│   │  • Ticket \#2206: "How to access downloaded file data in workflow"  
│   │  • Shows FileDownloadBlock → PDFParserBlock → SendEmailBlock chain  
│   │  • Demonstrates output parameter passing (key gap in current docs)  
│  
│  
└── Job Application Pipeline  
    \- Apply to multiple jobs from a list of URLs  
    \- Customize cover letter per job using TextPromptBlock  
    \- Handle different application forms (Lever, Greenhouse, LinkedIn)  
    \- Track success/failure via webhooks  
    \- Avoid duplicate applications  
    │  
    │  WHY THIS COOKBOOK:  
    │  • Job application is \#1 demo but only shows single application  
    │  • Real users want to apply to 50+ jobs at scale  
    │  • Shows ForLoop \+ Conditional \+ TextPrompt together  
    │  • Webhook integration for external tracking systems

\+ Include the existing ‘Skyvern in Action’ page. We can even show building these in the UI.