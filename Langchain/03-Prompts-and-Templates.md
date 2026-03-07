# 03 — Prompts and Templates

## Why Templates?

Hardcoding messages doesn't scale. Templates let you inject variables at runtime.

## PromptTemplate — Single Text, No Roles

For simple prompts without system messages:

```python
from langchain_core.prompts import PromptTemplate

dynamic_prompt = PromptTemplate.from_template("Write a fun fact about {topic}")

# Fill in the variable
ready_prompt = dynamic_prompt.invoke({"topic": "honey"})

# Send to LLM
llm.invoke(ready_prompt).content
```

- Creates one plain text prompt
- No concept of system/human/ai roles
- Uses `.from_template("single string")`

## ChatPromptTemplate — Multiple Messages with Roles

For when you need a system message (tone/personality):

```python
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a {tone} assistant"),
    ("human", "Write a fun fact about {topic}")
])

# Fill in ALL variables
ready = prompt.invoke({"tone": "funny", "topic": "space"})

# Send to LLM
llm.invoke(ready).content
```

- Creates a list of messages with roles
- Uses `.from_messages([list of tuples])`
- Each tuple = `(role, content)`

## Key Differences

| | `PromptTemplate` | `ChatPromptTemplate` |
|---|---|---|
| Method | `.from_template("string")` | `.from_messages([tuples])` |
| Roles | No roles | system, human, ai |
| Use when | Simple questions, no personality | Need to set AI behavior |

## Variables Must Match

The `{variable}` names in your template MUST exactly match the keys in your dictionary:

```python
# Template has {city} and {question}
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are from {city}"),
    ("human", "{question}")
])

# Dictionary keys must match
prompt.invoke({"city": "Bangalore", "question": "What's up?"})  # ✅
prompt.invoke({"place": "Bangalore", "input": "What's up?"})    # ❌ wrong keys
```

## Multiple Variables

You can have as many variables as you want:

```python
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a {tone} assistant from {city}"),
    ("human", "Tell me about {topic} in {language}")
])

prompt.invoke({
    "tone": "funny",
    "city": "Mumbai", 
    "topic": "cricket",
    "language": "Hindi"
})
```

## Mixing Tuples and Message Classes

You can mix both styles:

```python
from langchain_core.messages import SystemMessage

prompt = ChatPromptTemplate.from_messages([
    SystemMessage(content="You are helpful"),  # fixed, no variables
    ("human", "Tell me about {topic}")          # dynamic
])
```

Note: `SystemMessage(content="...")` is fixed — you can't put `{variables}` in it.

## Quick Reference

```python
# Simple prompt, no roles
PromptTemplate.from_template("Tell me about {topic}")

# Chat prompt with roles
ChatPromptTemplate.from_messages([
    ("system", "You are {role}"),
    ("human", "{question}")
])

# Single human message shortcut
ChatPromptTemplate.from_template("Tell me a joke about {topic}")
```

---

Previous: [[02-Messages]] | Next: [[04-Structured-Output]]
