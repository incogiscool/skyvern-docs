# Workflows

A workflow is a sequence of blocks that Skyvern executes in order. Each block does one thing - navigate a website, extract data, make an HTTP request, run a loop - and passes its output to the next block. When you need to automate a multi-step process that's too complex for a single task (like logging in, navigating to a report, downloading a file, and emailing it), workflows are the right tool.

This page explains how to structure workflows, what block types are available, how to pass data between blocks, and how to deploy and run them.

## Workflow structure

A workflow is a YAML document with three sections:

```yaml
title: My Workflow
description: Logs in and downloads a report
workflow_definition:
  version: 1
  parameters:
    - ...   # inputs and secrets
  blocks:
    - ...   # the automation steps
  finally_block_label: cleanup  # optional: always-run teardown block
```

Blocks execute top-to-bottom by default. You can override this with `next_block_label` on any block, or use a conditional block to branch based on runtime conditions.

Every block needs a unique `label`. Labels must be valid Python identifiers (letters, digits, underscores; no spaces or hyphens). Skyvern automatically sanitizes labels when importing a workflow, but it's better to write clean labels from the start.

## Parameters

Parameters are the workflow's inputs. You define them at the top, and blocks reference them with Jinja2 templates: `{{ my_param }}`.

There are several parameter types. The most common is `workflow` - a plain value passed in when you run the workflow:

```yaml
parameters:
  - parameter_type: workflow
    key: target_url
    workflow_parameter_type: string
    default_value: "https://example.com"

  - parameter_type: workflow
    key: max_items
    workflow_parameter_type: integer
    default_value: 10
```

When you run the workflow via the API, you pass parameter values in the `parameters` field:

```json
{
  "workflow_id": "wpid_abc123",
  "parameters": {
    "target_url": "https://my-site.com",
    "max_items": 25
  }
}
```

To use a stored credential (like a password), use a `credential` parameter:

```yaml
parameters:
  - parameter_type: credential
    key: login_creds
    credential_id: cred_abc123
```

When the workflow runs, `login_creds` is populated with the decrypted credential data from the vault.

## Block outputs and templating

Every block produces an output accessible as `{{ label_output }}` in downstream blocks. For example, if you have a block labeled `fetch_items`, its output is `{{ fetch_items_output }}`.

This is how you chain blocks together:

```yaml
blocks:
  - block_type: task
    label: fetch_items
    url: "{{ target_url }}"
    data_extraction_goal: "Extract all product names and prices"
    data_schema:
      type: array
      items:
        type: object
        properties:
          name: {type: string}
          price: {type: number}

  - block_type: text_prompt
    label: summarize
    prompt: |
      Summarize the following product list in plain English:
      {{ fetch_items_output }}
```

Jinja2 templating is available in any string field of any block. The workflow engine evaluates templates at runtime, so `{{ fetch_items_output }}` becomes the actual data extracted by the first block before the second block runs.

By default, the template engine is in `lax` mode - undefined variables resolve to empty strings rather than throwing an error. If you need strict validation, set `WORKFLOW_TEMPLATING_STRICTNESS=strict` in your environment.

## Block types

### task

Navigates a website and optionally extracts data. This is the core block for any browser automation.

```yaml
- block_type: task
  label: login_step
  url: "https://example.com/login"
  navigation_goal: "Log in with the provided credentials"
  parameter_keys: [login_creds]
  max_retries: 2
  max_steps_per_run: 15
```

Key fields:
- `url` - the starting URL (can be a template)
- `navigation_goal` - what to do on the page
- `data_extraction_goal` - what to extract (optional)
- `data_schema` - JSON Schema for the extracted data
- `parameter_keys` - which parameters to make available to the agent
- `complete_criterion` - custom completion condition
- `terminate_criterion` - when to stop with failure

### for_loop

Iterates over a list, running a set of inner blocks for each item. The loop variable is available inside inner blocks as `{{ current_value }}` and `{{ current_index }}`.

```yaml
- block_type: for_loop
  label: process_each_url
  loop_over_parameter_key: url_list
  loop_blocks:
    - block_type: task
      label: scrape_page
      url: "{{ current_value }}"
      data_extraction_goal: "Extract the page title and main heading"
```

The `loop_over_parameter_key` must reference a parameter whose value is a list. Use `complete_if_empty: true` to skip the loop entirely if the list is empty.

### conditional

Branches to different blocks based on a condition. Each branch has a `criteria` with a Jinja2 expression or a prompt, and a `next_block_label` pointing to the block to run when the condition is true. One branch can be marked `is_default: true` to act as the else case.

```yaml
- block_type: conditional
  label: check_status
  branch_conditions:
    - criteria:
        criteria_type: jinja2_template
        expression: "{{ fetch_output.status == 'active' }}"
      next_block_label: process_active
      description: "Handle active items"
    - is_default: true
      next_block_label: handle_inactive
      description: "Handle everything else"
```

### code

Executes arbitrary Python code in a sandboxed environment. Useful for data transformation, calculations, or logic that's hard to express in a prompt.

```yaml
- block_type: code
  label: transform_data
  code: |
    items = fetch_output
    result = [{"name": x["name"].upper(), "total": x["price"] * 1.1} for x in items]
```

The output of a code block is whatever is assigned to `result` inside the code.

### text_prompt

Sends a prompt to an LLM (without a browser) and returns the response. Good for summarization, classification, or generating text based on data from previous blocks.

```yaml
- block_type: text_prompt
  label: classify_intent
  prompt: |
    Classify the following message as: complaint, question, or feedback.
    Return only the category name.

    Message: {{ user_message }}
```

### http_request

Makes an HTTP request and returns the response body. Use this to call external APIs, send data to other services, or fetch data without a browser.

```yaml
- block_type: http_request
  label: notify_slack
  method: POST
  url: "https://hooks.slack.com/services/..."
  headers:
    Content-Type: application/json
  body:
    text: "Workflow completed: {{ summarize_output }}"
```

### send_email

Sends an email via SMTP. Configure SMTP credentials in environment variables (`SMTP_HOST`, `SMTP_PORT`, `SMTP_USERNAME`, `SMTP_PASSWORD`).

```yaml
- block_type: send_email
  label: email_report
  sender: "automation@company.com"
  recipients: ["team@company.com"]
  subject: "Daily Report - {{ today }}"
  body: "{{ report_output }}"
```

### navigation, extraction, login, action

These are focused task variants that set `navigation_goal`, `data_extraction_goal`, and similar fields for you. They work identically to `task` but make intent clearer in the YAML.

```yaml
- block_type: login
  label: sign_in
  url: "{{ login_url }}"
  parameter_keys: [credentials]

- block_type: extraction
  label: get_data
  url: "{{ data_url }}"
  data_extraction_goal: "Extract all invoice rows"
  data_schema:
    type: array
```

### wait

Pauses the workflow for a fixed number of seconds. Useful when waiting for an external process to complete.

```yaml
- block_type: wait
  label: pause
  wait_sec: 30
```

### file_download

Navigates to a URL and downloads a file.

```yaml
- block_type: file_download
  label: download_report
  url: "{{ report_url }}"
  navigation_goal: "Click the Export button and download the CSV"
```

### pdf_parser

Parses a PDF file (from a URL or previous download) and extracts its text content.

### print_page

Prints the current page to a PDF artifact.

### http_request (webhook)

Identical to `http_request` but idiomatically used for outbound webhook calls.

### workflow_trigger

Triggers another workflow and optionally waits for it to complete. Useful for composing smaller reusable workflows.

```yaml
- block_type: workflow_trigger
  label: run_sub_workflow
  workflow_permanent_id: wpid_xyz789
  payload:
    input_data: "{{ current_value }}"
  wait_for_completion: true
```

## The finally block

The `finally_block_label` at the workflow level specifies a block that always runs when the workflow ends - whether it succeeded, failed, or was canceled. Use it for cleanup, notifications, or logging.

```yaml
workflow_definition:
  finally_block_label: send_status_email
  blocks:
    - block_type: task
      label: main_work
      ...
    - block_type: send_email
      label: send_status_email
      ...
```

The finally block runs even if an earlier block fails, as long as the workflow itself reaches a terminal state.

## Creating and running a workflow

**Create the workflow:**

```bash
curl -X POST https://api.skyvern.com/api/v1/workflows \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "yaml_definition": "title: My Workflow\nworkflow_definition:\n  version: 1\n  parameters: []\n  blocks:\n    - ..."
  }'
```

The response includes the `workflow_id` (starts with `wpid_`).

**Run the workflow:**

```bash
curl -X POST https://api.skyvern.com/api/v1/run/workflows \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "workflow_id": "wpid_abc123",
    "parameters": {"target_url": "https://example.com"}
  }'
```

Poll `GET /runs/{run_id}` to check status and retrieve output.

## A complete example

This workflow logs in to a site, extracts a list of orders, and emails a summary:

```yaml
title: Order Summary Workflow
description: Logs in, extracts orders, sends summary email
workflow_definition:
  version: 1
  parameters:
    - parameter_type: workflow
      key: site_url
      workflow_parameter_type: string
    - parameter_type: credential
      key: site_creds
      credential_id: cred_abc123
    - parameter_type: workflow
      key: recipient_email
      workflow_parameter_type: string

  blocks:
    - block_type: login
      label: sign_in
      url: "{{ site_url }}/login"
      parameter_keys: [site_creds]

    - block_type: extraction
      label: get_orders
      url: "{{ site_url }}/orders"
      data_extraction_goal: "Extract all orders from the last 7 days"
      data_schema:
        type: array
        items:
          type: object
          properties:
            order_id: {type: string}
            amount: {type: number}
            status: {type: string}

    - block_type: text_prompt
      label: summarize_orders
      prompt: |
        Write a brief 3-sentence summary of these orders for an executive:
        {{ get_orders_output }}

    - block_type: send_email
      label: send_report
      sender: "automation@company.com"
      recipients: ["{{ recipient_email }}"]
      subject: "Weekly Order Summary"
      body: "{{ summarize_orders_output }}"
```
