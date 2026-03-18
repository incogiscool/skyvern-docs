# Documentation Writer's Guide

**Status**: 2 of 13 pages written. 11 pages remaining.

## Quick Start for Writers

1. Read this entire file (10 min)
2. Read **context/context-index.md** Overview & Architecture (20 min)
3. Pick your assigned page from the list below
4. Follow the template for that page type
5. Reference context-index.md liberally using Ctrl+F
6. Use file paths to verify details in context/skyvern/

---

## Pages Status

### ✅ Complete (2 pages)

| Page | File | Status | Writer |
|------|------|--------|--------|
| getting-started | docs/getting-started.md | ✅ Written | Manicule |
| architecture | docs/architecture.md | ✅ Written | Manicule |

### 🟡 To Do (11 pages)

| # | Page | Type | Priority | Key Topics |
|---|------|------|----------|-----------|
| 1 | api-reference | Reference | HIGH | All REST endpoints, request/response schemas, error handling |
| 2 | sdk-guide | Guide | HIGH | Python SDK (Skyvern class, decorators), TypeScript SDK, workflow DSL |
| 3 | workflows | Guide | HIGH | Workflow syntax, block types, parameters, variables |
| 4 | agents-and-ai | Explanation | MEDIUM | Agent orchestration, LLM providers, prompt engineering |
| 5 | browser-automation | Guide | MEDIUM | AI locators, vision capabilities, session management |
| 6 | credentials-and-auth | Guide | MEDIUM | Credential storage, JWT, multi-tenancy, webhooks |
| 7 | data-extraction | Guide | MEDIUM | Data schemas, JSON, extraction patterns, examples |
| 8 | error-handling | Guide | MEDIUM | Common errors, debugging, retry strategies, timeouts |
| 9 | deployment | Guide | MEDIUM | Docker Compose, cloud setup, env vars, migrations |
| 10 | cli-reference | Reference | LOW | Command reference, options, config files |
| 11 | examples | Reference | LOW | Code snippets, common workflows, multi-page scenarios |

---

## How to Use context-index.md

### Structure

The context-index.md is organized in these sections:

| Section | Lines | What It Contains | Use For |
|---------|-------|------------------|---------|
| Overview | 1-30 | Project summary, tech stack | Understanding what Skyvern is |
| File Tree | 34-150 | Where everything is located | Finding source files to reference |
| Architecture | 152-250 | System design, data flow, component interactions | Understanding how things work together |
| Key Concepts | 252-420 | Task, Step, Workflow, Block, Artifact definitions | Understanding domain models |
| API Surface | 422-480 | All REST endpoints grouped by route file | Reference for API docs |
| Data Model | 482-600 | Database models, relationships, enums | Understanding data schema |
| Configuration | 630-680 | All environment variables | Config reference page |
| Dependencies | 682-700 | External packages and libraries | Dependency reference |
| Build & Run | 702-750 | How to build and test | Deployment and setup docs |
| Patterns & Conventions | 752-800 | Code style, error handling, naming | Helping developers understand patterns |

### Search Strategy

**When writing about [TOPIC], search context-index.md for:**

| Topic | Search Terms |
|-------|--------------|
| Task execution | "Task", "Agent", "task flow", "step generation" |
| Workflows | "Workflow", "block", "WorkflowService", "WorkflowRunContext" |
| Browser control | "Playwright", "Browser", "SkyvernBrowser", "vision" |
| LLMs | "LLM", "OpenAI", "Anthropic", "Vertex", "prompt" |
| Database | "SQLAlchemy", "TaskModel", "StepModel", "organization_id" |
| Authentication | "JWT", "auth", "credentials", "multi-tenancy" |
| APIs | "FastAPI", "route", "endpoint", "schema" |
| Configuration | "environment variable", "config", "Settings" |

---

## Page Templates

### Reference Pages (api-reference, cli-reference, examples)

**Purpose**: Help users look up specific details quickly.

**Structure**:
```markdown
# [Page Title]

Brief intro (1-2 sentences)

## Section 1: Category
- Item 1
- Item 2
  - Sub-item
  - Code example

## Section 2: Category
...
```

**Tips**:
- Use tables for lists of endpoints/commands
- Include example requests and responses
- Group related items together
- Add curl/Python examples for API reference
- Cross-reference other pages

**For api-reference.md specifically**:
- Group endpoints by resource (tasks, workflows, browser sessions, etc.)
- Show request/response examples with real data
- Include HTTP status codes
- Document required headers and authentication
- Show optional parameters

**For cli-reference.md specifically**:
- List all commands (from context/skyvern/skyvern/cli/)
- Show syntax: `skyvern <command> [options]`
- Describe each option
- Include examples for each command
- Document exit codes

**For examples.md specifically**:
- Group by use case (form filling, data extraction, login, etc.)
- Provide complete runnable examples
- Include both Python SDK and CLI examples
- Show common patterns and anti-patterns
- Document error cases

### Guide Pages (getting-started, sdk-guide, workflows, browser-automation, credentials-and-auth, data-extraction, error-handling, deployment)

**Purpose**: Help users learn and accomplish specific tasks.

**Structure**:
```markdown
# [Page Title]

Brief intro explaining what you'll learn (2-3 sentences)

## Prerequisites / Quick Start
[What users need before following]

## Section 1: Core Concept
[Explain idea + show example]

## Section 2: How to Use
[Step-by-step instructions]

## Section 3: Advanced / Tips
[Edge cases, optimization, best practices]

## Troubleshooting / Common Issues
[FAQ + solutions]
```

**Tips**:
- Start simple, progress to advanced
- Use code examples liberally
- Include "why" not just "how"
- Show common mistakes
- Link to related pages
- Use callout boxes for important notes

**For sdk-guide.md specifically**:
- Show how to import and use Skyvern class
- Document all decorators (@workflow, @action, @cached, @login)
- Explain workflow DSL syntax
- Show Python SDK examples
- Brief intro to TypeScript SDK
- Cross-reference workflows.md for complex examples

**For workflows.md specifically**:
- Explain workflow definition format (YAML? JSON? Python?)
- Document each block type (Action, Validation, Loop, etc.)
- Show how to pass parameters and variables
- Explain conditional logic and loops
- Provide examples of nested blocks
- Link to examples.md for real-world workflows

**For browser-automation.md specifically**:
- Explain AI-powered locators vs. CSS selectors
- Show screenshot analysis and vision LLM calls
- Document session management
- Handle dynamic content and loading
- Show waiting strategies
- Explain accessibility locators

**For credentials-and-auth.md specifically**:
- Explain credential types supported
- Show how to store/retrieve credentials
- Document JWT authentication
- Explain multi-tenancy model
- Show webhook security (signatures)
- Link to deployment for env var setup

**For data-extraction.md specifically**:
- Explain extraction goal format
- Show JSON schema definition
- Document complex nested data handling
- Provide use case examples (e-commerce, forms, tables)
- Show validation and error handling
- Link to examples.md for real workflows

**For error-handling.md specifically**:
- List common errors and causes
- Show debugging techniques
- Explain retry strategies
- Document timeout configuration
- Show logging and debug mode
- Provide troubleshooting flowcharts

**For deployment.md specifically**:
- Show Docker Compose setup
- Document cloud deployment options
- List all key environment variables
- Explain database migrations
- Show monitoring setup
- Document scaling considerations

### Explanation Pages (architecture, agents-and-ai)

**Already written**: architecture.md is complete.

**Purpose**: Help readers understand *why* and *how*, not just *what*.

**Structure**:
```markdown
# [Page Title]

Intro with diagram or mental model (2-3 sentences)

## Concept 1: [Big Idea]
[Explain the idea, why it matters, how it works]
[Include ASCII diagram if helpful]

## Concept 2: [Another Big Idea]
...

## How They Work Together
[Tie concepts together with concrete example]

## Design Rationale
[Why was it designed this way?]
```

**For agents-and-ai.md specifically**:
- Explain the agent loop (perception → decision → action)
- Document LLM provider support and selection
- Show prompt engineering approach
- Explain token counting and cost tracking
- Document vision LLM usage
- Link to api-reference.md for LLM configuration

---

## Critical Resources in context-index.md

### For API Reference
- **Location**: context-index.md lines 422-480
- **Contains**: All endpoint paths, route files, response models
- **Example**: Task endpoints in `forge/sdk/routes/tasks.py`

### For SDK Guide
- **Location**: context-index.md lines 252-280 (Skyvern class)
- **Contains**: SDK classes, decorators, workflow DSL
- **Example**: `@workflow` decorator in `skyvern/workflow/__init__.py`

### For Workflows
- **Location**: context-index.md lines 370-420 (Block types)
- **Contains**: Block definitions, nested structure, execution logic
- **Example**: ActionBlock in `forge/sdk/workflow/models/`

### For Browser Automation
- **Location**: context-index.md lines 300-330 (Vision & AI)
- **Contains**: Vision LLM calls, screenshot analysis, locators
- **Example**: Vision module in `forge/sdk/api/llm/`

### For Credentials & Auth
- **Location**: context-index.md lines 330-360 (Auth & Security)
- **Contains**: JWT, multi-tenancy, credential storage
- **Example**: Models in `forge/sdk/db/models.py` (CredentialModel)

### For Data Extraction
- **Location**: context-index.md lines 310-325 (Extraction)
- **Contains**: Goal format, extraction patterns
- **Example**: Schema in `forge/sdk/schemas/tasks.py`

### For Configuration
- **Location**: context-index.md lines 630-680
- **Contains**: All env variables with descriptions
- **Example**: Settings class in `skyvern/config.py`

### For Deployment
- **Location**: context-index.md lines 702-750 (Build & Run)
- **Contains**: Docker, database setup, migrations
- **Example**: docker-compose.yml in context/skyvern/

---

## Writing Best Practices

### 1. Code Examples

**DO**:
```python
from skyvern import Skyvern

# Clear, focused example
skyvern = Skyvern(api_key="your-key")
result = skyvern.run_task(url="https://example.com", goal="Find the price")
```

**DON'T**:
```python
# Vague or incomplete
from skyvern import *
# ... lots of setup code ...
skyvern.run_task(...)  # without explanation
```

**Guidelines**:
- Make examples self-contained and runnable
- Show realistic use cases
- Include error handling
- Comment non-obvious parts
- Show both success and failure cases

### 2. Cross-References

**DO**:
```markdown
See [Workflows](/docs/workflows.md) for the complete syntax.
Refer to [API Reference](/docs/api-reference.md#task-endpoints) for all parameters.
```

**DON'T**:
```markdown
See the other docs for more info.
For API details, check the code.
```

**Guidelines**:
- Link to related pages
- Use markdown links with proper paths
- Include anchor links for long pages
- Explain what readers will find

### 3. Diagrams

**Good for**:
- Data flow (task → agent → browser → database)
- Component interaction
- State transitions
- Architecture relationships

**Tools**:
- ASCII art (simple, renders everywhere)
- Markdown tables (for comparisons)
- Callout boxes for emphasis

### 4. Tone & Voice

- **Clear**: Assume reader is unfamiliar with Skyvern but knows web automation
- **Practical**: Show "how to do it", not just theory
- **Honest**: Acknowledge limitations and edge cases
- **Helpful**: Anticipate questions and pain points

### 5. Structure

- **Headings**: Use H2-H4 (# for page title only)
- **Lists**: Use bullets or numbers appropriately
- **Tables**: Group related information
- **Callouts**: Warning ⚠️, Note 📝, Tip 💡, Example 📌
- **Code blocks**: Specify language (python, bash, json)

---

## Checklist Before Submitting

- [ ] Read context-index.md sections relevant to your page
- [ ] Followed the template for your page type (Reference/Guide/Explanation)
- [ ] Verified file paths by checking context/skyvern/
- [ ] Included code examples that run
- [ ] Cross-referenced related pages
- [ ] Proofread for clarity and tone
- [ ] Used consistent markdown formatting
- [ ] Tested links (if applicable)
- [ ] Checked that no section is longer than 2,000 words
- [ ] Included examples and use cases

---

## Common Patterns to Document

### Pattern: Running a Task

**Explain in**: getting-started.md (already done), sdk-guide.md

```python
from skyvern import Skyvern

skyvern = Skyvern(api_key="...")
result = skyvern.run_task(url="...", goal="...")
```

### Pattern: Using Workflows

**Explain in**: workflows.md, examples.md

```yaml
steps:
  - type: action
    action: click
    target: button[type='submit']
  - type: validation
    expected: success message
```

### Pattern: Error Handling

**Explain in**: error-handling.md

```python
try:
    result = skyvern.run_task(...)
except TimeoutError:
    # Handle timeout
except FailedStepError:
    # Handle step failure
```

### Pattern: Extracting Data

**Explain in**: data-extraction.md

```python
result = skyvern.run_task(
    url="...",
    goal="...",
    extraction_schema={"properties": {...}}
)
data = result.extraction
```

---

## Getting Help

If you're stuck on a section:

1. **Search context-index.md** for the topic
2. **Check open-questions.md** to see if it's noted as unclear
3. **Look at the source code** in context/skyvern/ (paths are in context-index.md)
4. **Check completed pages** (getting-started.md, architecture.md) for style and structure

---

## Submitting Your Work

1. Write your page in docs/[page-name].md
2. Test all code examples run
3. Verify all links work
4. Commit: `git add docs/[page-name].md && git commit -m "draft: [page-name]"`
5. Push to your branch
6. Create PR for review

---

**Last Updated**: 2026-03-18
**Total Pages**: 13 (2 complete, 11 to do)
**Context Size**: 836 lines in context-index.md
**Source Code**: All in context/skyvern/

