# 08 — Conditional Chains

## What Are Conditional Chains?

Instead of running all branches (parallel), only ONE branch runs based on a condition.

```
                        condition: positive?
                        ┌→ YES → LinkedIn chain
Movie review → classify │
                        └→ NO  → Instagram chain
```

## RunnableBranch

```python
from langchain_core.runnables import RunnableBranch

conditional_chain = RunnableBranch(
    (lambda x: "Positive" in x, linkedIn_chain),   # if positive → LinkedIn
    instagram_runnable                                # else → Instagram (default)
)
```

- First argument(s): tuples of `(condition_function, chain_to_run)`
- Last argument: the default/fallback chain (no condition)

## Full Example: Review Classifier

```python
from pydantic import BaseModel
from typing import Literal
from langchain_core.runnables import RunnableLambda, RunnableBranch

# Step 1: Schema to force Positive/Negative
class ReviewSchema(BaseModel):
    movie_summary_flag: Literal["Positive", "Negative"]

# Step 2: LLM with structured output
llm_structured = llm.with_structured_output(ReviewSchema)

# Step 3: Extract the flag as a string
def extract_flag(input: ReviewSchema) -> str:
    return input.model_dump()["movie_summary_flag"]

flag_runnable = RunnableLambda(extract_flag)

# Step 4: Define the conditional routing
conditional_chain = RunnableBranch(
    (lambda x: "Positive" in x, linkedIn_chain),
    instagram_runnable  # default
)

# Step 5: Full orchestration
final_chain = (
    movie_prompt 
    | llm_structured 
    | flag_runnable 
    | conditional_chain
)

final_chain.invoke({"input": "I loved this KGF movie"})
```

## How the Lambda Works

```python
lambda x: "Positive" in x
```

- `x` = the input the branch receives (in this case, the string `"Positive"` or `"Negative"`)
- If `"Positive"` is in `x`, returns `True` → runs `linkedIn_chain`
- Otherwise → falls to the default (`instagram_runnable`)

## Known Gotcha

In this pattern, the **original review text gets lost**. The conditional branches receive just the word `"Positive"` or `"Negative"` — not the actual review. The branches end up generating posts about the classification label, not the content. See [[Tips-and-Common-Doubts]] for more on this.

## Conditional Chains vs If/Else

Conditional chains using `RunnableBranch` are essentially fancy if/else statements. You could achieve the same with plain Python:

```python
classification = classify_chain.invoke(input)
if classification == "Positive":
    result = linkedIn_chain.invoke(input)
else:
    result = instagram_chain.invoke(input)
```

`RunnableBranch` keeps it within the LCEL chain structure — cleaner for complex pipelines.

---

Previous: [[07-Parallel-Chains]] | Next: [[09-Tools-and-Binding]]
