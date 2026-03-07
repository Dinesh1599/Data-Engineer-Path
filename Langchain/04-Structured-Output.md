# 04 — Structured Output

## The Problem

Without structure, LLM returns a blob of text:

```
"Here's a joke! Why did the scarecrow win an award? Because he was outstanding in his field!"
```

You can't easily separate the setup from the punchline programmatically.

## The Solution: Pydantic BaseModel

Define a schema (structure) that the LLM MUST follow:

```python
from pydantic import BaseModel, Field

class JokeSchema(BaseModel):
    setup: str = Field(description="The setup for the joke")
    punchline: str = Field(description="The punchline for the joke")
```

Then bind it to the LLM:

```python
llm_structured = llm.with_structured_output(JokeSchema)
result = llm_structured.invoke("Tell me a joke")

result.setup      # "Why did the scarecrow win an award?"
result.punchline  # "Because he was outstanding in his field"
```

Now the LLM is forced to return exactly two fields. No more, no less.

## What is `Field`?

`Field(description="...")` gives the LLM a hint about what to put in each field:

```python
# Without Field — LLM guesses what goes where
class Schema(BaseModel):
    s: str
    p: str

# With Field — LLM knows exactly what you want
class Schema(BaseModel):
    s: str = Field(description="The setup for the joke")
    p: str = Field(description="The punchline for the joke")
```

Not required, but very helpful — especially when field names are vague.

## What is `.model_dump()`?

Converts a Pydantic object into a plain dictionary:

```python
result = llm_structured.invoke("Tell me a joke")

result              # JokeSchema(setup="...", punchline="...")  → Pydantic object
result.model_dump() # {"setup": "...", "punchline": "..."}     → plain dict
```

Useful when you need to pass data to something that only accepts dicts (like JSON serialization or prompt templates).

## Pydantic vs TypedDict

Both define structure, but they work differently:

### Pydantic BaseModel (recommended for LangChain)

```python
from pydantic import BaseModel

class JokeSchema(BaseModel):
    setup: str
    punchline: str

# Returns a Pydantic object → access with dot notation
result.setup
result.punchline

# Validates strictly — wrong key? ERROR
# Wrong type? ERROR
```

### TypedDict (lighter, less strict)

```python
from typing import TypedDict

class JokeSchema(TypedDict):
    setup: str
    punchline: str

# Returns a plain dict → access with bracket notation
result["setup"]
result["punchline"]

# No runtime validation — wrong key? Still works (just a warning)
```

### When to use which

| | Pydantic | TypedDict |
|---|---|---|
| Validation | Strict (runtime errors) | Soft (type hints only) |
| Access style | `result.setup` | `result["setup"]` |
| `Field(description=)` | Yes | Needs `Annotated` (clunky) |
| Use for | LLM structured output | Simple type hints in regular code |

**Rule of thumb:** Use Pydantic for LLM output. Use TypedDict for regular Python dicts.

## Using `Literal` for Fixed Options

Force the LLM to pick from specific values:

```python
from typing import Literal

class ReviewSchema(BaseModel):
    sentiment: Literal["Positive", "Negative"]
```

Now the LLM can ONLY return "Positive" or "Negative" — nothing else.

## Pydantic Warning (Safe to Ignore)

You might see this warning:

```
PydanticSerializationUnexpectedValue(Expected `none` - serialized value may not be as expected...)
```

This is just Pydantic being cautious during internal serialization. Your code works fine. Suppress it if it bothers you:

```python
import warnings
warnings.filterwarnings("ignore", category=UserWarning)
```

---

Previous: [[03-Prompts-and-Templates]] | Next: [[05-LCEL-and-Chains]]
