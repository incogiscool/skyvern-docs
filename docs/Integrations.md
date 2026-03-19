# Integrations

Skyvern connects to external tools through dedicated integration packages and a Model Context Protocol (MCP) server. This page covers the four available integrations: LangChain, LlamaIndex, Make, and MCP.

## LangChain

The `skyvern-langchain` package wraps Skyvern tasks as [LangChain tools](https://python.langchain.com/docs/modules/tools/), so any LangChain agent can trigger browser automation.

**Install:**

```bash
pip install skyvern-langchain
pip install langchain-openai  # or your preferred LangChain LLM package
```

**Basic usage (via Skyvern Cloud API):**

```python
import asyncio
from skyvern_langchain.client import RunTask

run_task = RunTask(api_key="YOUR_SKYVERN_API_KEY")

async def main():
    result = await run_task.ainvoke(
        "Navigate to the Hacker News homepage and get the top 3 posts."
    )
    print(result)

asyncio.run(main())
```

**As a LangChain agent tool:**

```python
import asyncio
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI
from langchain.agents import initialize_agent, AgentType
from skyvern_langchain.client import RunTask, GetTask

load_dotenv()

tools = [RunTask(), GetTask()]
agent = initialize_agent(
    tools=tools,
    llm=ChatOpenAI(model="gpt-4o"),
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True,
)

asyncio.run(agent.arun(
    "Navigate to Hacker News and return the top 3 posts. "
    "Then check the task status until it's complete."
))
```

Available tool classes: `RunTask` (synchronous, blocks until done), `DispatchTask` (async, returns immediately), `GetTask` (retrieve task by ID).

For local mode (without an API key), import from `skyvern_langchain.agent` instead of `skyvern_langchain.client`, and run `skyvern init` first.

## LlamaIndex

The `skyvern-llamaindex` package provides a `SkyvernTool` that wraps Skyvern tasks as LlamaIndex tools.

**Install:**

```bash
pip install skyvern-llamaindex
```

**Usage:**

```python
from dotenv import load_dotenv
from llama_index.agent.openai import OpenAIAgent
from llama_index.llms.openai import OpenAI
from skyvern_llamaindex.agent import SkyvernTool

load_dotenv()

skyvern_tool = SkyvernTool()

agent = OpenAIAgent.from_tools(
    tools=[skyvern_tool.run_task()],
    llm=OpenAI(model="gpt-4o"),
    verbose=True,
)

response = agent.chat(
    "Run a task with Skyvern: navigate to Hacker News and get the top 3 posts."
)
print(response)
```

The `SkyvernTool` class provides: `run_task()` (synchronous), `dispatch_task()` (async), `get_task()` (retrieve by ID).

## Make (formerly Integromat)

Skyvern has a [Make](https://www.make.com) module that lets you trigger Skyvern tasks and workflows from Make scenarios without writing code. The module is available in the Make marketplace.

The `integrations/make/contact_form_make.com_blueprint.json` file in the repository contains an example Make blueprint that demonstrates calling the Skyvern API to fill out a contact form. Import it into Make to see a working example.

To connect Make to Skyvern:
1. Add the Skyvern app in your Make scenario
2. Create a connection using your Skyvern API key
3. Choose the "Run Task" or "Run Workflow" action
4. Map your scenario data to the task parameters

## MCP (Model Context Protocol)

The MCP integration exposes Skyvern as a tool server that MCP-enabled AI applications (Claude Desktop, Cursor, Windsurf, and others) can call directly to automate browser tasks.

**Install and configure:**

```bash
pip install skyvern
skyvern init  # runs the setup wizard, configures your MCP app
```

The `skyvern init` wizard automatically writes the MCP configuration for Cursor, Windsurf, and Claude Desktop. For any other MCP-compatible application, add this config manually:

```json
{
  "mcpServers": {
    "Skyvern": {
      "env": {
        "SKYVERN_BASE_URL": "https://api.skyvern.com",
        "SKYVERN_API_KEY": "YOUR_SKYVERN_API_KEY"
      },
      "command": "/path/to/python",
      "args": ["-m", "skyvern", "run", "mcp"]
    }
  }
}
```

For local mode, set `SKYVERN_BASE_URL` to `http://localhost:8000` and start the server first with `skyvern run server`.

Once configured, your AI assistant can browse the web, fill out forms, download files, and research information on demand. The MCP server runs Skyvern tasks in response to natural language requests from the AI application.
