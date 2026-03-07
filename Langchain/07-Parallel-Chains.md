# 07 — Parallel Chains

## What Are Parallel Chains?

Sometimes you need one output to feed into multiple chains that run simultaneously.

```
                    ┌→ LinkedIn chain → LinkedIn post
Movie summary →     │
                    └→ Instagram chain → Instagram post
```

## RunnableParallel

```python
from langchain_core.runnables import RunnableParallel

linkedIn_chain = linkedIN_prompt | llm | StrOutputParser()
instagram_chain = instagram_prompt | llm | StrOutputParser()

final_chain = (
    movie_prompt 
    | llm 
    | StrOutputParser() 
    | dict_runnable 
    | RunnableParallel(branches={
        "linkedIn": linkedIn_chain,
        "instagram": instagram_chain
    })
)

result = final_chain.invoke({"question": "KGF"})
```

## The Output Structure

The result comes back as a nested dict:

```python
{
    "branches": {
        "linkedIn": "LinkedIn post content here...",
        "instagram": "Instagram post content here..."
    }
}
```

The `"branches"` key comes from the parameter name in `RunnableParallel(branches={...})`. The inner keys (`"linkedIn"`, `"instagram"`) are what you defined.

## Beautifying the Output

Add a function at the end to clean up:

```python
def beautify(result: dict) -> dict:
    linkedin = result["branches"]["linkedIn"]
    instagram = result["branches"]["instagram"]
    return {"linkedIn": linkedin, "instagram": instagram}

beautify_runnable = RunnableLambda(beautify)

beautified_chain = final_chain | beautify_runnable
```

## Mixing Chain Styles in Parallel

You can mix regular chains and function-based chains:

```python
# Regular chain
linkedIn_chain = linkedIN_prompt | llm | StrOutputParser()

# Function-based chain
def insta_chain(text: dict):
    chain = instagram_prompt | llm | StrOutputParser()
    return chain.invoke(text)

instagram_runnable = RunnableLambda(insta_chain)

# Both work in RunnableParallel
RunnableParallel(branches={
    "linkedIn": linkedIn_chain,        # regular chain
    "instagram": instagram_runnable     # function-based
})
```

## Using `()` for Multi-line Chains

Wrapping in parentheses is just a Python formatting trick for readability — no functional difference:

```python
# Hard to read — one long line
final_chain = movie_prompt | llm | StrOutputParser() | dict_runnable | RunnableParallel(...)

# Easy to read — wrapped in ()
final_chain = (
    movie_prompt 
    | llm 
    | StrOutputParser() 
    | dict_runnable 
    | RunnableParallel(...)
)
```

## Async for True Parallelism

`RunnableParallel` only runs truly in parallel with async:

```python
# Sync — branches run one after another
result = final_chain.invoke(input)

# Async — branches run at the same time (faster)
result = await final_chain.ainvoke(input)
```

In Jupyter notebooks, `await` works directly. In scripts, use `asyncio.run()`.

---

Previous: [[06-Custom-Runnables]] | Next: [[08-Conditional-Chains]]
