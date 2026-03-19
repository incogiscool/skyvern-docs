# TypeScript SDK

The `@skyvern/client` package is the official TypeScript/JavaScript client for the Skyvern API. It runs in Node.js and browser environments, and is auto-generated from the OpenAPI spec using Fern.

## Installation

```bash
npm install @skyvern/client
# or
yarn add @skyvern/client
```

Current version: `1.0.24`

## Creating a client

```typescript
import { SkyvernClient } from "@skyvern/client";

const client = new SkyvernClient({
  apiKey: "your-api-key",
});
```

By default, the client connects to Skyvern Cloud (`https://api.skyvern.com`). To point at a local or self-hosted instance:

```typescript
import { SkyvernClient, SkyvernEnvironment } from "@skyvern/client";

// Local
const client = new SkyvernClient({
  apiKey: "your-api-key",
  environment: SkyvernEnvironment.Local,  // http://localhost:8000
});

// Custom URL
const client = new SkyvernClient({
  apiKey: "your-api-key",
  baseUrl: "https://my-skyvern.example.com",
});
```

**Constructor options:**

| Option | Type | Description |
|---|---|---|
| `apiKey` | string | Your Skyvern API key (sent as `x-api-key` header) |
| `environment` | SkyvernEnvironment | `Cloud`, `Staging`, or `Local` |
| `baseUrl` | string | Override the base URL entirely |
| `timeoutInSeconds` | number | Default request timeout (seconds) |
| `maxRetries` | number | Number of automatic retries on failure |
| `abortSignal` | AbortSignal | Default abort signal for all requests |

## Running tasks

```typescript
const taskRun = await client.runTask({
  body: {
    prompt: "Find the top 3 posts on Hacker News",
  },
});

console.log(taskRun.run_id);   // "tsk_v2_abc123"
console.log(taskRun.status);   // "queued"
```

With structured output:

```typescript
const taskRun = await client.runTask({
  body: {
    prompt: "Get the top 5 posts on Hacker News",
    data_extraction_schema: {
      type: "array",
      items: {
        type: "object",
        properties: {
          title: { type: "string" },
          score: { type: "integer" },
          url: { type: "string" },
        },
      },
    },
    proxy_location: "RESIDENTIAL",
    max_steps: 10,
  },
});
```

**Request body fields** (inside `body`):

| Field | Type | Default | Description |
|---|---|---|---|
| `prompt` | string | required | Natural language goal |
| `url` | string | `null` | Starting URL |
| `engine` | string | `skyvern-2.0` | Execution engine |
| `data_extraction_schema` | object | `null` | JSON Schema for structured output |
| `proxy_location` | string/object | `RESIDENTIAL` | Proxy region |
| `max_steps` | number | `null` | Step limit |
| `webhook_url` | string | `null` | URL to POST results to |
| `browser_session_id` | string | `null` | Reuse a persistent session |
| `totp_identifier` | string | `null` | TOTP email/phone |
| `totp_url` | string | `null` | URL for TOTP polling |
| `browser_address` | string | `null` | CDP URL for external browser |

## Polling for results

`runTask` and `runWorkflow` return immediately with a `run_id`. Poll `getRun` to check completion:

```typescript
async function waitForRun(runId: string): Promise<GetRunResponse> {
  const terminalStatuses = new Set([
    "completed", "failed", "terminated", "timed_out", "canceled"
  ]);

  while (true) {
    const run = await client.getRun(runId);
    if (terminalStatuses.has(run.status)) {
      return run;
    }
    await new Promise((resolve) => setTimeout(resolve, 5000));
  }
}

const taskRun = await client.runTask({ body: { prompt: "..." } });
const result = await waitForRun(taskRun.run_id);
console.log(result.output);
```

## Running workflows

```typescript
const workflowRun = await client.runWorkflow({
  body: {
    workflow_id: "wpid_abc123",
    parameters: {
      target_url: "https://example.com",
      max_items: 10,
    },
  },
});

const result = await waitForRun(workflowRun.run_id);
```

## Canceling a run

```typescript
await client.cancelRun("tsk_abc123");
```

## Working with workflows

```typescript
// List workflows
const workflows = await client.getWorkflows({
  page_size: 20,
});

// Get a specific workflow
const workflow = await client.getWorkflow("wpid_abc123");

// Create a workflow from YAML
const newWorkflow = await client.createWorkflow({
  body: {
    yaml_definition: `
title: My Workflow
workflow_definition:
  version: 1
  parameters: []
  blocks:
    - block_type: task
      label: scrape
      url: https://example.com
      navigation_goal: Extract all headings
    `,
  },
});

// Delete a workflow
await client.deleteWorkflow({ workflowId: "wpid_abc123" });
```

## Credentials

```typescript
// Create a password credential
const credential = await client.createCredential({
  body: {
    name: "Amazon Login",
    credential_type: "password",
    credential: {
      username: "user@example.com",
      password: "mypassword",
    },
  },
});
console.log(credential.credential_id);

// Create a credit card credential
const card = await client.createCredential({
  body: {
    name: "Company Visa",
    credential_type: "credit_card",
    credential: {
      card_number: "4111111111111111",
      card_cvv: "123",
      card_exp_month: "12",
      card_exp_year: "2027",
      card_brand: "visa",
      card_holder_name: "Jane Smith",
    },
  },
});

// List credentials
const creds = await client.getCredentials({ page_size: 50 });

// Delete a credential
await client.deleteCredential({ credentialId: "cred_abc123" });

// Send a TOTP code for 2FA
await client.sendTotpCode({
  content: "Your code is 847291",
  totp_identifier: "user@example.com",
  task_id: "tsk_abc123",
});
```

## Artifacts

```typescript
// Get a specific artifact
const artifact = await client.getArtifact("art_abc123");
console.log(artifact.signed_url);  // Download URL (valid 24h)

// Get all artifacts for a run
const artifacts = await client.getRunArtifacts({
  runId: "tsk_abc123",
  artifact_type: ["recording", "screenshot_action"],
});
```

## Browser sessions

```typescript
// Create a persistent browser session
const session = await client.createBrowserSession({
  body: {
    proxy_location: "RESIDENTIAL",
    timeout: 60,
  },
});

// Use it in a task
const taskRun = await client.runTask({
  body: {
    prompt: "Check my order status",
    browser_session_id: session.browser_session_id,
  },
});

// List active sessions
const sessions = await client.getBrowserSessions();

// Close a session
await client.closeBrowserSession({ browserSessionId: session.browser_session_id });
```

## Login and download helpers

The client exposes two high-level methods that map to pre-built workflow templates:

```typescript
// Log in using a stored credential
const loginRun = await client.login({
  body: {
    credential_id: "cred_abc123",
    url: "https://example.com/login",
  },
});

// Download files from a website
const downloadRun = await client.downloadFiles({
  navigation_goal: "Click the Export button and download the CSV file",
  url: "https://example.com/reports",
});
```

## Scripts

Scripts are accessed via the nested `scripts` resource:

```typescript
// Run a script
await client.scripts.runScript("s_abc123");
```

## Error handling

The client throws typed error classes you can catch:

```typescript
import { SkyvernError, SkyvernTimeoutError } from "@skyvern/client";

try {
  const run = await client.runTask({ body: { prompt: "..." } });
} catch (error) {
  if (error instanceof SkyvernTimeoutError) {
    console.error("Request timed out");
  } else if (error instanceof SkyvernError) {
    console.error(`HTTP ${error.statusCode}: ${JSON.stringify(error.body)}`);
  } else {
    throw error;
  }
}
```

| Error class | When thrown |
|---|---|
| `SkyvernError` | HTTP errors (400, 404, 500, etc.) |
| `SkyvernTimeoutError` | Request timed out |
| `BadRequestError` | 400 response |
| `NotFoundError` | 404 response |
| `UnprocessableEntityError` | 422 validation error |

## Per-request options

Every method accepts an optional `requestOptions` object for request-level overrides:

```typescript
const run = await client.runTask(
  { body: { prompt: "..." } },
  {
    timeoutInSeconds: 120,
    maxRetries: 3,
    apiKey: "override-key",
    abortSignal: controller.signal,
  }
);
```

## Node.js vs browser

The SDK works in both environments. In Node.js, it uses the built-in `fetch`. In the browser, it also uses `fetch`. No environment-specific shims are needed.

For browser use, never include your API key in client-side code. Proxy requests through your backend instead.

## Full working example

```typescript
import { SkyvernClient } from "@skyvern/client";

const client = new SkyvernClient({ apiKey: process.env.SKYVERN_API_KEY! });

async function main() {
  // Submit a task
  const taskRun = await client.runTask({
    body: {
      prompt: "Extract the top 5 posts from Hacker News",
      data_extraction_schema: {
        type: "array",
        items: {
          type: "object",
          properties: {
            title: { type: "string" },
            score: { type: "integer" },
          },
        },
      },
    },
  });

  console.log(`Task started: ${taskRun.run_id}`);

  // Poll until complete
  const terminalStatuses = new Set(["completed", "failed", "terminated", "timed_out", "canceled"]);
  let result = await client.getRun(taskRun.run_id);

  while (!terminalStatuses.has(result.status)) {
    await new Promise((r) => setTimeout(r, 5000));
    result = await client.getRun(taskRun.run_id);
  }

  console.log(`Status: ${result.status}`);
  console.log(`Output:`, result.output);
}

main().catch(console.error);
```
