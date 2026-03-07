# 10 — ReAct Agents

## What is ReAct?

**ReAct = Reasoning + Acting.** The LLM thinks, takes action, observes the result, and repeats until it has a final answer.

```
User question
    ↓
LLM thinks: "I need to search for this"
    ↓
Calls tool (acting)
    ↓
Gets tool result (observation)
    ↓
LLM thinks: "Do I have enough info?"
    ↓
NO → calls another tool (loops back)
YES → returns final answer to user
```

This loop is what makes agents "autonomous" — the LLM decides on its own which tools to use and when to stop.

## Creating a ReAct Agent

```python
from langchain.agents import create_agent

agent = create_agent(llm, toolkit)
```

That's it. Two arguments: the model and the tools. Everything else (tool binding, the loop, tool execution) is handled automatically.

## Invoking the Agent

```python
result = agent.invoke({
    "messages": [("user", "What's the latest stock market news?")]
})

result["messages"][-1].pretty_print()
```

The agent always expects `{"messages": [...]}` as input.

### Message Formats (All Work)

```python
# Tuple style
{"messages": [("user", "your question")]}

# Dict style
{"messages": [{"role": "user", "content": "your question"}]}

# Message class style
from langchain_core.messages import HumanMessage
{"messages": [HumanMessage(content="your question")]}
```

## Streaming (See Progress in Real-Time)

```python
events = agent.stream(
    {"messages": [("user", "What's the latest stock market news?")]},
    stream_mode="values"
)

for event in events:
    event["messages"][-1].pretty_print()
```

This shows you each step as it happens: tool calls, tool results, and the final answer.

### Stream Modes

| Mode | What it shows |
|---|---|
| `"values"` | Full state after each step (most common) |
| `"updates"` | Only what changed in each step |
| `"messages"` | Token-by-token output (ChatGPT typing effect) |
| `"custom"` | Your own custom progress updates |
| `"debug"` | Everything — for debugging |

You can combine modes: `stream_mode=["values", "messages"]`

## All Invocation Methods

| Method | Description |
|---|---|
| `.invoke()` | Get full response at once |
| `.stream()` | See each step as it happens |
| `.ainvoke()` | Async invoke |
| `.astream()` | Async stream |

## What Happens Under the Hood

When you call `agent.invoke()`, it runs the manual tool-calling loop from [[09-Tools-and-Binding]] automatically:

1. Sends your message to the LLM
2. LLM says "I want to call tool X with args Y"
3. Agent runs the tool
4. Sends tool result back to LLM
5. LLM evaluates: need more info? → loop back to step 2
6. LLM satisfied → returns final answer

## Adding a System Prompt

```python
# Simple string
agent = create_agent(llm, toolkit, prompt="You are a helpful financial analyst")

# Or format with ChatPromptTemplate first
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a {role}")
])
formatted = prompt.invoke({"role": "financial analyst"})
agent = create_agent(llm, toolkit, prompt=formatted.messages[0])
```

A system prompt is optional — the agent works without one.

## Why Can't LCEL Build Agents?

LCEL is for **linear** flows: A → B → C → done.

Agents are **loops**: A → B → C → back to A → B → done.

The looping/decision-making behavior can't be expressed with `|` pipes. That's why agents use `create_agent` (backed by LangGraph) instead of LCEL.

---

Previous: [[09-Tools-and-Binding]] | Next: [[11-SQL-Database-Agent]]
