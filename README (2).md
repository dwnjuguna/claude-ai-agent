# 🤖 Claude AI Agent with Web Search

A simple AI agent built with Python and the Anthropic SDK that can search the web and answer questions using real-time information.

---

## Prerequisites

- Python 3.10+
- An [Anthropic API key](https://console.anthropic.com)

---

## Installation

### 1. Install dependencies

```bash
pip3 install anthropic ddgs
```

### 2. Set your API key

Replace `your-key-here` with your actual Anthropic API key:

```bash
export ANTHROPIC_API_KEY="your-key-here"
```

---

## Setup

Create a project folder and navigate into it:

```bash
mkdir my-agent
cd my-agent
```

---

## The Agent Code

Create a file called `agent.py` with the following code:

```python
import anthropic
import json
from ddgs import DDGS

client = anthropic.Anthropic()

# --- Define your tools ---
tools = [
    {
        "name": "web_search",
        "description": "Search the web for current information on a topic.",
        "input_schema": {
            "type": "object",
            "properties": {
                "query": {
                    "type": "string",
                    "description": "The search query"
                }
            },
            "required": ["query"]
        }
    }
]

# --- Real search using DuckDuckGo ---
def run_tool(name, inputs):
    if name == "web_search":
        query = inputs["query"]
        with DDGS() as ddgs:
            results = list(ddgs.text(query, max_results=3))
            return json.dumps(results)
    return "Tool not found."

# --- The agent loop ---
def run_agent(user_message):
    print(f"\nUser: {user_message}\n")
    messages = [{"role": "user", "content": user_message}]

    while True:
        response = client.messages.create(
            model="claude-opus-4-5",
            max_tokens=1024,
            tools=tools,
            messages=messages
        )

        if response.stop_reason == "tool_use":
            tool_results = []
            for block in response.content:
                if block.type == "tool_use":
                    print(f"🔧 Claude is using tool: {block.name} with input: {block.input}")
                    result = run_tool(block.name, block.input)
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": result
                    })
            messages.append({"role": "assistant", "content": response.content})
            messages.append({"role": "user", "content": tool_results})

        elif response.stop_reason == "end_turn":
            for block in response.content:
                if hasattr(block, "text"):
                    print(f"Claude: {block.text}")
            break

# --- Run it! ---
run_agent("What are the latest developments in AI?")
```

---

## Running the Agent

```bash
python3 agent.py
```

### Example Output

```
User: What are the latest developments in AI?

🔧 Claude is using tool: web_search with input: {'query': 'latest developments in AI 2025'}
Claude: Based on the latest search results, here are some of the most notable recent developments in AI: ...
```

---

## How It Works

The agent runs in a simple loop:

1. You give Claude a question
2. Claude decides whether to call the web search tool
3. Your code runs the search and returns real results
4. Claude reads the results and formulates an answer
5. The loop ends when Claude is done

---

## Changing the Question

To ask the agent something different, update the last line of `agent.py`:

```python
run_agent("Your question here")
```

---

## Next Steps

- Add more tools (calculator, file reader, weather API)
- Accept questions interactively from the Terminal
- Build a simple web UI with [Streamlit](https://streamlit.io)
- Store conversation history for multi-turn conversations

---

## Resources

- [Anthropic Documentation](https://docs.anthropic.com)
- [Anthropic API Console](https://console.anthropic.com)
- [Tool Use Guide](https://docs.anthropic.com/en/docs/build-with-claude/tool-use)
