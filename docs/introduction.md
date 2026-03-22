# Introduction

Skyvern is an open-source browser automation platform that uses vision LLMs and computer vision to operate websites the same way a person would. Instead of fragile XPath selectors or site-specific scripts, you give Skyvern a plain-English goal and a URL, and it figures out what to click, type, and navigate on its own. Because it reasons from what it sees in the viewport rather than from hard-coded DOM paths, it works on websites it has never encountered before and keeps working when those sites redesign their layouts.

## How Skyvern works

When you run a task, Skyvern opens a real browser with Playwright, takes a screenshot of the page, and sends that screenshot to a vision LLM alongside your goal. The LLM identifies the relevant elements and decides what actions to take next: click a button, fill a field, scroll, wait. Skyvern executes those actions, takes another screenshot, and repeats the loop until the goal is complete or the task fails.

This is fundamentally different from traditional automation tools. There are no pre-defined selectors to maintain. If a site moves a button or renames a form field, the agent just looks at the new page and figures out what changed. The same prompt that fills out a contact form on one site can fill out a structurally different one on another.

## Choose your path

Skyvern is available in three forms. Pick the one that matches how you work:

**Skyvern Cloud (app.skyvern.com)** is the managed web UI for building and running automations without writing code. You get a visual workflow editor, a library of templates, run history with recordings, and built-in credential management. This is a good fit for operations teams who want to automate repetitive browser work without involving a developer for every change.

**API and Python SDK** is how developers integrate Skyvern into their own systems. You install the `skyvern` package, get an API key from app.skyvern.com, and start automating with a few lines of Python. The browser runs on Skyvern's infrastructure by default, so there's nothing to provision. You can also point the agent at a browser running on your own machine when you need to access local networks or watch what's happening in real time.

**Self-hosted deployment** runs the full Skyvern stack on your own infrastructure using Docker Compose. You bring your own LLM API keys and your data never leaves your network. This is the right choice for regulated industries, high-volume workloads, or anywhere data residency matters. It requires more setup and you take on responsibility for scaling and maintenance.

---

> **Scope of this documentation:** This site covers the **API and Python SDK**. For the no-code Cloud UI, visit [app.skyvern.com](https://app.skyvern.com). For infrastructure setup, see [Self-Hosted Deployment](self-hosted-deployment.md).

Ready to run your first automation? Continue to the [Quickstart](quickstart.md).
