# Browser Automation

Skyvern's browser layer sits between the LLM agent and the web. This page explains how Skyvern manages browser instances, what session types are available, how to configure proxies and geolocation, and what options exist for connecting to an external browser.

## How the browser works

Skyvern uses [Playwright](https://playwright.dev/) as its browser automation library, running Chromium in headful mode by default. Every task or workflow gets a browser context — an isolated environment with its own cookies, local storage, and network state, similar to an incognito window.

When a task starts, Skyvern:
1. Creates a new Playwright browser context (or reuses an existing persistent session)
2. Opens a page and navigates to the target URL
3. Takes a screenshot and sends it to the LLM
4. Executes the action the LLM returns (click, type, scroll, etc.)
5. Repeats until the task is complete or the step limit is reached
6. Closes the browser context and saves artifacts

Anti-bot detection is countered by running Chrome with `--disable-blink-features=AutomationControlled` and `--disable-extensions-except` flags, and by ignoring the `--enable-automation` default arg. This removes the common Playwright detection signatures without requiring a separate stealth library.

## Browser types

The `BROWSER_TYPE` config setting controls which browser mode is used:

**`chromium-headful`** (default) — A local Chrome/Chromium instance managed by Playwright. The browser opens on screen (or in a virtual display in Docker). This is the standard mode for most deployments.

**`cdp-connect`** — Connect to an existing Chrome instance via the Chrome DevTools Protocol. Useful if you want Skyvern to control a browser that's already running with your personal profile, extensions, and saved logins.

To use CDP mode, start Chrome with remote debugging enabled:
```bash
# macOS
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
  --remote-debugging-port=9222 \
  --user-data-dir=/tmp/chrome-cdp-profile

# Windows
chrome.exe --remote-debugging-port=9222 --user-data-dir="C:\chrome-cdp-profile"
```

Then set these in your `.env`:
```
BROWSER_TYPE=cdp-connect
BROWSER_REMOTE_DEBUGGING_URL=http://127.0.0.1:9222
```

Or pass the CDP URL per-task using the `browser_address` field:
```json
{
  "prompt": "Download my latest invoice",
  "browser_address": "http://127.0.0.1:9222"
}
```

## Persistent browser sessions

By default, each task creates a fresh browser context that is destroyed when the task ends. **Persistent browser sessions** let you save the browser state (cookies, local storage, authentication tokens) and reuse it across multiple tasks or runs.

This is useful when:
- You want to log in once and reuse the session for many tasks, avoiding repeated login steps
- You need to maintain state across a multi-stage workflow that runs over time
- You're connecting Skyvern Cloud to a browser running on your machine

### Creating a persistent session

```bash
curl -X POST https://api.skyvern.com/api/v1/browser-sessions \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json"
```

The response includes a `browser_session_id` (starts with `pbs_`). Pass this ID in any task or workflow run request:

```json
{
  "prompt": "Go to the dashboard and export the weekly report",
  "browser_session_id": "pbs_abc123"
}
```

The task picks up the browser exactly where it left off. Sessions persist until you explicitly close them or they time out.

### Connecting your local browser to Skyvern Cloud

If you want Skyvern Cloud to control a Chrome instance running on your local machine (useful for sites behind a VPN or where you're already logged in), you can expose it via a tunnel:

```bash
skyvern browser serve --tunnel
```

This starts Chrome and creates an ngrok tunnel. The command prints a URL like `https://abc123.ngrok-free.dev`. Use that as the `browser_address` in your task:

```json
{
  "prompt": "Download the latest invoice from my account",
  "browser_address": "https://abc123.ngrok-free.dev"
}
```

Always protect tunneled browsers with an API key (`skyvern browser serve --tunnel --api-key YOUR_KEY`) — without it, anyone with the URL can control your browser.

## Proxy and geolocation

Many websites serve different content based on geographic location, or block traffic from cloud datacenters. Skyvern supports residential proxies and geo-targeting to handle this.

### Proxy locations

Pass `proxy_location` in any task or workflow run request. The accepted values are region codes:

```json
{"proxy_location": "RESIDENTIAL"}
```

Common values:
- `RESIDENTIAL` — US residential IP (default)
- `US-CA`, `US-NY`, `US-TX`, `US-FL`, `US-WA` — specific US states
- `RESIDENTIAL_GB`, `RESIDENTIAL_DE`, `RESIDENTIAL_FR`, `RESIDENTIAL_ES` — European countries
- `RESIDENTIAL_JP`, `RESIDENTIAL_IN`, `RESIDENTIAL_AU`, `RESIDENTIAL_BR` — Asia-Pacific / Americas
- `NONE` — no proxy

When a proxy location is set, the browser's timezone is automatically configured to match the region. For example, `US-CA` sets the timezone to `America/Los_Angeles`.

### GeoTarget (granular targeting)

For city or state-level targeting, use a `GeoTarget` object instead of a string:

```json
{
  "proxy_location": {
    "country": "US",
    "subdivision": "CA",
    "city": "San Francisco"
  }
}
```

Supported fields:
- `country` — ISO 3166-1 alpha-2 code (required). Must be in the [supported list](https://www.skyvern.com/docs): US, GB, DE, FR, JP, AU, BR, CA, and others.
- `subdivision` — ISO 3166-2 code without country prefix (e.g., `CA` for California, `ENG` for England).
- `city` — City name in English from GeoNames.

### Enabling proxies in self-hosted deployments

If you're running Skyvern locally, you supply your own proxy pool:

```
ENABLE_PROXY=true
HOSTED_PROXY_POOL=http://username:password@proxy1.example.com:8080,http://username:password@proxy2.example.com:8080
```

Skyvern picks a random proxy from the pool for each browser context.

## Browser configuration options

These settings in `.env` control the browser environment:

| Setting | Default | Description |
|---|---|---|
| `BROWSER_TYPE` | `chromium-headful` | `chromium-headful` or `cdp-connect` |
| `BROWSER_WIDTH` | `1920` | Viewport width in pixels |
| `BROWSER_HEIGHT` | `1080` | Viewport height in pixels |
| `BROWSER_TIMEZONE` | `America/New_York` | Default timezone (overridden by proxy location) |
| `BROWSER_LOCALE` | `null` | Browser locale (e.g., `en-US`) |
| `BROWSER_ACTION_TIMEOUT_MS` | `5000` | Timeout for individual browser actions |
| `BROWSER_LOADING_TIMEOUT_MS` | `60000` | Timeout for page loads |
| `BROWSER_SCREENSHOT_TIMEOUT_MS` | `20000` | Timeout for taking screenshots |
| `BROWSER_MAX_PAGES_NUMBER` | `10` | Maximum number of tabs per browser context |
| `BROWSER_ADDITIONAL_ARGS` | `[]` | Additional Chrome command-line flags |
| `DOWNLOAD_PATH` | `./downloads` | Where downloaded files are saved |
| `VIDEO_PATH` | `./video` | Where session recordings are saved |
| `HAR_PATH` | `./har` | Where HAR (HTTP archive) files are saved |

## Real-time browser streaming

You can watch a task or workflow run in real time via WebSocket. The UI uses this to show the live browser view as Skyvern works.

Connect to the VNC WebSocket stream for a task:
```
ws://localhost:8000/api/v1/stream/vnc/task/{task_id}?apikey=YOUR_API_KEY
```

For a persistent browser session:
```
ws://localhost:8000/api/v1/stream/vnc/browser_session/{browser_session_id}?apikey=YOUR_API_KEY
```

The stream carries a standard VNC protocol feed that any VNC-compatible viewer can display. The Skyvern UI renders this directly in the browser.

## Recording and artifacts

Every task produces a video recording (`recording` artifact type) and a set of action screenshots (`screenshot_action`). These are stored in S3 or Azure Blob Storage and accessible via signed URLs with 24-hour expiration. HAR files (network request logs) and Playwright trace files are also available for debugging.

Retrieve all artifacts for a run:
```
GET /api/v1/runs/{run_id}/artifacts
```
