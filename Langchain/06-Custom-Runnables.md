# 06 — Custom Runnables

## The Problem

You can't put a regular Python function in an LCEL chain:

```python
def make_uppercase(text):
    return text.upper()

# ❌ Won't work — regular functions don't have .invoke()
chain = prompt | llm | StrOutputParser() | make_uppercase
```

## The Solution: RunnableLambda

Wrap your function to make it chain-compatible:

```python
from langchain_core.runnables import RunnableLambda

def make_uppercase(text):
    return text.upper()

# ✅ Works
chain = prompt | llm | StrOutputParser() | RunnableLambda(make_uppercase)
```

`RunnableLambda` gives your function an `.invoke()` method so it fits in the chain.

## The Dictionary Maker Pattern

This is a very common pattern. Prompt templates need dicts, but the previous step outputs a string.

**The problem:**
```
StrOutputParser → outputs "some text" (string)
ChatPromptTemplate → needs {"text": "some text"} (dict)
```

**The fix — a dictionary maker function:**

```python
def to_dict(text: str) -> dict:
    return {"text": text}

dict_runnable = RunnableLambda(to_dict)

# Now the chain works
chain = (
    prompt1 | llm | StrOutputParser() 
    | dict_runnable  # converts string → dict
    | prompt2 | llm | StrOutputParser()
)
```

**Important:** The key in the returned dict (`"text"`) must match the `{variable}` in the next template.

## Writing a Full Chain as a Function

You can put an entire chain inside a function and wrap it with RunnableLambda:

```python
def instagram_chain(text: dict):
    chain_insta = instagram_prompt | llm | StrOutputParser()
    return chain_insta.invoke(text)

instagram_runnable = RunnableLambda(instagram_chain)
```

Benefits:
- More control over intermediate steps
- Can make changes in-between
- Don't need the dictionary maker — handle it inside the function

## Chains Are Already Runnables

Important: any chain you create is a runnable by default. No need to wrap it:

```python
chain_a = prompt | llm | StrOutputParser()

# You can directly connect it to another step
full_chain = chain_a | dict_runnable | chain_b
# chain_a is already a runnable — no RunnableLambda needed
```

Only regular Python functions need `RunnableLambda`.

## What Each Step Should Return

Always return what the **next step** expects:

| Next step is... | Return... |
|---|---|
| A prompt template | A dict with matching keys |
| An LLM | Messages or a PromptValue |
| StrOutputParser | An AIMessage |
| Another RunnableLambda | Whatever that function expects |

---

Previous: [[05-LCEL-and-Chains]] | Next: [[07-Parallel-Chains]]
