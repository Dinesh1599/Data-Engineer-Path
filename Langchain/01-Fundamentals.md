# 01 — Fundamentals

## What is an AI Agent?

An AI agent = LLM + Tools. That's it.

- An **LLM** on its own can only generate text
- When you give it **tools** (functions it can call), it becomes an agent that can actually DO things — send emails, query databases, search the web
- **Agentic AI** = multiple AI agents talking to each other in a workflow (orchestration)

If you're a data engineer, think of it as a pipeline/DAG where each node is an AI-powered task.

## Why LangChain?

Without LangChain, you'd use separate SDKs for each model:

```python
# Without LangChain — different SDK for each provider
from openai import OpenAI       # OpenAI SDK
from anthropic import Anthropic  # Anthropic SDK
from google import genai         # Google SDK
```

With LangChain, one SDK handles everything:

```python
# With LangChain — one interface for all models
from langchain_openai import ChatOpenAI
from langchain_anthropic import ChatAnthropic
# Same .invoke() method for all
```

LangChain = a wrapper that gives you a unified interface to all LLM providers. Pre-built classes, error handling, and boilerplate code — all done for you.

## Your First LLM Call

### Without LangChain (raw OpenAI SDK)

```python
from openai import OpenAI
import os

client = OpenAI(api_key=os.environ.get("OPENAI_API_KEY"))
response = client.chat.completions.create(
    model="gpt-5-mini",
    messages=[{"role": "user", "content": "Tell me a fun fact"}]
)
print(response.choices[0].message.content)
```

### With LangChain

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-5-mini", temperature=0)
llm.invoke("Tell me a fun fact").content
```

### With init_chat_model (newer way)

```python
from langchain.chat_models import init_chat_model

llm = init_chat_model(model="gpt-5-mini", temperature=0.7)
llm.invoke("Tell me a fun fact").content
```

`init_chat_model` auto-detects the provider from the model name. No need to import provider-specific classes.

## What is `temperature`?

- `temperature=0` → deterministic, best answer, no creativity
- `temperature=0.7-0.9` → more creative, varied responses
- Use 0 for serious/factual tasks, higher for creative tasks

## The `.content` Thing

`llm.invoke()` returns an `AIMessage` object, not a plain string:

```python
response = llm.invoke("Tell me a joke")
# AIMessage(content="Why did...", response_metadata={...}, usage_metadata={...})

response.content  # "Why did..." — just the text
```

You'll use `.content` a lot. Later, `StrOutputParser()` will do this automatically in chains.

## Environment Setup (Quick Reference)

```python
from dotenv import load_dotenv
import os

load_dotenv()  # loads variables from .env file

if os.environ.get("OPENAI_API_KEY"):
    print("Key Exists")
else:
    raise ValueError("Missing API KEY")
```

Your `.env` file:
```
OPENAI_API_KEY=sk-your-key-here
```

Always add `.env` to `.gitignore`. Never hardcode API keys in your code.

---

Previous: [[00-Index]] | Next: [[02-Messages]]
