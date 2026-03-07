# 05 — LCEL and Chains

## What is a Chain?

A chain = a sequence of tasks executed in order. If you're a data engineer, it's just a pipeline.

```
Task 1 → Task 2 → Task 3 → Output
```

## What is LCEL?

**LCEL = LangChain Expression Language.** It's the `|` (pipe) operator that connects steps together.

```python
chain = prompt | llm | StrOutputParser()
```

Read it left to right: prompt → llm → parser. Output of each step feeds into the next.

## How LCEL Works (Step by Step)

```python
chain = prompt | llm | StrOutputParser()
chain.invoke({"topic": "space"})
```

Behind the scenes, `chain.invoke()` calls `.invoke()` on EVERY step:

```python
# Step 1: prompt.invoke() fills in the template
result1 = prompt.invoke({"topic": "space"})

# Step 2: llm.invoke() sends to AI, gets AIMessage back
result2 = llm.invoke(result1)

# Step 3: StrOutputParser().invoke() extracts .content
result3 = StrOutputParser().invoke(result2)
```

The `|` automates this — you call `.invoke()` once, it runs all three.

## What is StrOutputParser?

It extracts the text string from the AIMessage object. It's a replacement for `.content`:

```python
# Without StrOutputParser — returns AIMessage object
chain = prompt | llm
chain.invoke({...})  # AIMessage(content="...", metadata={...})

# With StrOutputParser — returns just the string
chain = prompt | llm | StrOutputParser()
chain.invoke({...})  # "just the text"
```

**Don't use StrOutputParser with structured output.** `with_structured_output()` already returns a parsed Pydantic object, not an AIMessage.

## Complete Example

```python
from langchain.chat_models import init_chat_model
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

llm = init_chat_model(model="gpt-5-mini", temperature=0.7)

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a {role}"),
    ("human", "{question}")
])

chain = prompt | llm | StrOutputParser()

result = chain.invoke({"role": "comedian", "question": "Tell me a joke"})
# "Why did the chicken cross the road?..."
```

## Manual Way vs LCEL

```python
# Manual (the long way)
formatted = prompt.invoke({"topic": "space"})
response = llm.invoke(formatted)
text = response.content

# LCEL (the short way) — same result
chain = prompt | llm | StrOutputParser()
text = chain.invoke({"topic": "space"})
```

## RunnableSequence (Alternative Syntax)

The `|` pipe is actually shorthand for `RunnableSequence`:

```python
from langchain_core.runnables import RunnableSequence

# These are identical
chain = prompt | llm | StrOutputParser()
chain = RunnableSequence(prompt, llm, StrOutputParser())
```

Nobody uses `RunnableSequence` directly — just use `|`. But you might see it in error messages or docs.

## The Golden Rule of Chains

> The output of one step becomes the input of the next step.

This means:
- Prompt templates output messages → LLM accepts messages ✅
- LLM outputs AIMessage → StrOutputParser accepts AIMessage ✅
- StrOutputParser outputs string → next step must accept a string

If there's a mismatch, the chain breaks. Always check: **what does the next step expect?**

## When to Use LCEL vs Direct invoke

- **Direct invoke** → quick scripts, testing, simple one-off calls
- **LCEL chains** → production apps, reusable logic, multi-step workflows

---

Previous: [[04-Structured-Output]] | Next: [[06-Custom-Runnables]]
