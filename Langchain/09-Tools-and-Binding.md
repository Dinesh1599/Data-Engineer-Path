# 09 — Tools and Tool Binding

## Why Tools?

LLMs can only generate text. They can't:
- Search the web for live news
- Send emails
- Query your database
- Access anything they weren't trained on

**Tools = Python functions that give the LLM real-world capabilities.**

## Built-in Tools (LangChain Integrations)

LangChain has pre-built tools for many services:

```python
# DuckDuckGo search (free, no API key)
from langchain_community.tools import DuckDuckGoSearchRun
search_tool = DuckDuckGoSearchRun(description="Tool to search the web for news")

# Wikipedia
from langchain_community.tools import WikipediaQueryRun
from langchain_community.utilities import WikipediaAPIWrapper
wikipedia_tool = WikipediaQueryRun(
    api_wrapper=WikipediaAPIWrapper(),
    description="Tool to search Wikipedia"
)
```

Browse all integrations at: [LangChain Integrations](https://python.langchain.com/docs/integrations/)

## Custom Tools (Your Own Functions)

Use the `@tool` decorator:

```python
from langchain_core.tools import tool

@tool
def enterprise_tool(query: str) -> str:
    """This is a tool to send emails to employees."""
    # your email sending code here
    return "Email sent"
```

**Critical: The docstring (`"""..."""`) is required.** The LLM reads it to decide when to use the tool. No docstring = LLM doesn't know what the tool does.

## Toolkit (Group of Tools)

```python
toolkit = [search_tool, wikipedia_tool, enterprise_tool]
```

Just a Python list of tool objects. Nothing fancy.

## Tool Binding

Binding = telling the LLM "these tools exist, you can request to use them."

```python
llm_binded = llm.bind_tools(toolkit)
response = llm_binded.invoke("What's the latest stock market news?")
```

**Important: `bind_tools` does NOT execute tools.** The LLM just says "I want to use duckduckgo_search with this query." Nobody actually runs it.

The response looks like:

```python
AIMessage(
    content='',              # empty! no answer
    tool_calls=[             # LLM wants to call tools
        {'name': 'duckduckgo_search', 'args': {'query': 'stock market news today'}, 'id': 'call_abc123'},
        {'name': 'duckduckgo_search', 'args': {'query': 'S&P 500 today'}, 'id': 'call_def456'},
    ]
)
```

## Manual Tool Calling (What Agents Do Under the Hood)

```python
from langchain_core.messages import ToolMessage

# Step 1: LLM requests tool calls
response = llm_binded.invoke("What's the stock market news?")

# Step 2: YOU run the tools
results = []
for tool_call in response.tool_calls:
    tool = next(t for t in toolkit if t.name == tool_call["name"])
    result = tool.invoke(tool_call["args"])
    results.append(ToolMessage(content=result, tool_call_id=tool_call["id"]))

# Step 3: Send everything back to LLM for final answer
final = llm_binded.invoke([
    ("user", "What's the stock market news?"),
    response,       # the AI's tool call request
    *results        # the tool results
])

print(final.content)  # NOW you get the actual answer
```

### Breaking Down Step 2

- `response.tool_calls` = list of dicts the LLM requested
- `tool_call["name"]` = which tool the LLM wants (e.g., `"duckduckgo_search"`)
- `next(t for t in toolkit if t.name == tool_call["name"])` = find the matching tool object from your list
- `t.name` = the internal name of the tool (not your variable name)
- `tool.invoke(tool_call["args"])` = actually run the tool with the LLM's suggested arguments
- `ToolMessage` = a special message type that wraps tool results

### Breaking Down Step 3

The `*results` unpacks the list. Without `*`, you'd have a nested list:

```python
[msg1, msg2, [result1, result2]]   # ❌ without *
[msg1, msg2, result1, result2]     # ✅ with *
```

## Why Learn the Manual Way?

Agents (`create_react_agent`) do all three steps automatically. But knowing the manual flow helps you:
- Understand what agents do under the hood
- Debug when things go wrong
- Add custom logic between steps (validation, human approval, logging)

---

Previous: [[08-Conditional-Chains]] | Next: [[10-ReAct-Agents]]
