# 02 — Messages

## Three Types of Messages

Think of a conversation between two people, but one of them is an AI.

| Message Type | Who sends it | Purpose |
|---|---|---|
| **SystemMessage** | You (the developer) | Sets the tone/behavior for the AI |
| **HumanMessage** | The user | The question or request |
| **AIMessage** | The AI/LLM | The response |

**Real-world analogy:** System message is like telling a customer service agent "be polite and helpful" before they start taking calls. The agent (AI) then responds to every customer (human) with that tone.

## Using Message Classes

```python
from langchain_core.messages import HumanMessage, SystemMessage, AIMessage

messages = [
    SystemMessage(content="You are a Gen Z assistant who always answers in a fun way"),
    HumanMessage(content="Bro, tell me a fun fact")
]

llm.invoke(messages).content
```

## Key Points

- **SystemMessage** = sets the personality/rules for the AI. You control this.
- **HumanMessage** = what the user asks. The user controls this.
- **AIMessage** = what the AI responds. The AI controls this.
- You can have multiple back-and-forth messages in the list
- System message usually goes first, but it's not strictly required
- Without a system message, the AI uses its default personality

## Role Aliases

LangChain accepts multiple names for the same roles:

| LangChain style | OpenAI style | Both work |
|---|---|---|
| `"human"` | `"user"` | Same thing |
| `"ai"` | `"assistant"` | Same thing |
| `"system"` | `"system"` | Same |

```python
# These are identical
("human", "Tell me a joke")
("user", "Tell me a joke")
```

## When to Use Messages vs Prompts

- **Messages** = static, manual way. Good for simple one-off calls.
- **Prompts (Templates)** = dynamic way with variables. Good for reusable, production code.

We cover prompts next.

---

Previous: [[01-Fundamentals]] | Next: [[03-Prompts-and-Templates]]
