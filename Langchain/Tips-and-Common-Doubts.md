# Tips and Common Doubts

Every question and confusion that came up while learning LangChain, answered clearly.

---

## LCEL / Chains

### "What does `prompt | llm` actually mean?"

Take the output of `prompt` and feed it as input to `llm`. The `|` is just a shortcut for:

```python
result = prompt.invoke(input)
llm.invoke(result)
```

### "Does `chain.invoke()` call `.invoke()` on every pipe?"

Yes. When you call `chain.invoke()`, it internally calls `.invoke()` on each step in order, passing output to the next. One `.invoke()` triggers all of them.

### "Is LCEL only for prompt-based functions?"

No. LCEL works with anything that has `.invoke()`: prompts, LLMs, parsers, retrievers, custom functions (via `RunnableLambda`), and even other chains.

### "What's the difference between `chain.invoke()` and `chain.ainvoke()`?"

`ainvoke` is the async version. For `RunnableParallel`, `ainvoke` runs branches truly simultaneously (faster). In Jupyter, use `await chain.ainvoke(input)`. In scripts, use `asyncio.run()`.

### "Does wrapping a chain in `()` change anything?"

No. It's just Python syntax to break long lines across multiple lines for readability:

```python
chain = (prompt | llm | parser)  # same as:
chain = prompt | llm | parser
```

---

## Prompts

### "Do I use `from_template` or `from_messages`?"

- `from_template("single string")` — one message, no roles
- `from_messages([list of tuples])` — multiple messages with roles

If you need a system message, use `from_messages`. Common mistake: writing `from_template([...])` — this will error.

### "Must the role order be system → human → ai?"

No strict order, but the natural convention is: system first (sets behavior), then human/ai take turns. Putting human before system won't crash but might confuse the LLM.

### "Can I use `'user'` instead of `'human'`?"

Yes. `"user"` and `"human"` are aliases. `"assistant"` and `"ai"` are aliases. Only `"system"` stays the same.

### "How do I remember all the imports and syntax?"

You don't memorize it. Use autocomplete in your IDE, keep a cheat sheet, and look things up when needed. After 3-4 small projects, the common imports become muscle memory. The uncommon ones — everyone looks those up.

---

## Structured Output

### "What does `Field(description=...)` do?"

It gives the LLM a hint about what to put in each field. Not required, but helps — especially when field names are vague like `s` or `p`.

### "What is `.model_dump()`?"

Converts a Pydantic object to a plain dict:

```python
joke.model_dump()  # {"setup": "...", "punchline": "..."}
```

Useful for JSON serialization or passing to prompt templates.

### "When should I use Pydantic vs TypedDict?"

- **Pydantic**: strict validation, dot notation access (`result.setup`), use for LLM structured output
- **TypedDict**: no validation, bracket access (`result["setup"]`), use for regular Python type hints

### "That Pydantic serializer warning — should I worry?"

No. It's just Pydantic being cautious internally. Your code works fine. Suppress with:

```python
import warnings
warnings.filterwarnings("ignore", category=UserWarning)
```

### "If I use structured output (`with_structured_output`), do I also need `StrOutputParser`?"

No! `with_structured_output` already returns a parsed Pydantic object. Adding `StrOutputParser` would break the chain because it expects an AIMessage, not a Pydantic model.

---

## Custom Runnables

### "Why can't I put a regular function in a chain?"

Regular functions don't have `.invoke()`. `RunnableLambda` wraps your function to give it `.invoke()`, making it chain-compatible.

### "Does my custom function MUST return a dictionary?"

Only if the next step is a prompt template (which needs a dict to fill `{variables}`). If the next step is another `RunnableLambda`, return whatever that function expects.

### "What's `next()` doing in `next(t for t in toolkit if t.name == ...)`?"

It grabs the first item from a list that matches a condition. It's a shortcut for a for-loop with a break:

```python
# Same thing
for t in toolkit:
    if t.name == "duckduckgo_search":
        tool = t
        break
```

---

## Tools and Agents

### "If I use `bind_tools`, why don't I get an answer?"

`bind_tools` only tells the LLM "these tools exist." The LLM responds with tool REQUESTS, not answers. You need to actually run the tools and send results back. Agents (`create_agent`) do this automatically.

### "Is the manual tool-calling approach what LangGraph uses?"

Yes. LangGraph automates the same loop (LLM → check for tool calls → run tools → send results back → repeat) but structures it as a graph with nodes and edges.

### "Is a docstring required for custom tools?"

Yes! The LLM reads the docstring to understand what the tool does. Without it, the LLM won't know when to use your tool. Use triple quotes `"""..."""`, not `#` comments.

### "Are single-quote docstrings `'''..'''` valid?"

Yes, Python treats `"""..."""` and `'''...'''` the same. Convention is double quotes.

### "`t.name` vs `tool_call["name"]` — what's what?"

- `t.name` = the internal name of a tool object (e.g., `"duckduckgo_search"`)
- `tool_call["name"]` = what the LLM requested (also `"duckduckgo_search"`)
- The comparison matches what the LLM wants with what you have

Note: `t.name` is NOT the Python variable name. Your variable might be `search_tool` but `search_tool.name` could be `"duckduckgo_search"`.

### "What does `*results` do in the invoke call?"

It unpacks a list into individual items:

```python
results = [ToolMessage1, ToolMessage2]
[msg, response, *results]  # → [msg, response, ToolMessage1, ToolMessage2]
[msg, response, results]   # → [msg, response, [ToolMessage1, ToolMessage2]]  ← nested, bad!
```

### "What does `**` do?"

Unpacks a dict into keyword arguments: `**{"a": 1, "b": 2}` → `a=1, b=2`

---

## Agents and Streaming

### "Is `agent.stream()` the only way to invoke an agent?"

No. You also have `.invoke()` (full response), `.ainvoke()` (async), and `.astream()` (async streaming). `.stream()` is most common for agents because it shows each step in real-time.

### "Can I use `ChatPromptTemplate` inside `agent.invoke()`?"

Not directly inside `.invoke()`. Format the prompt first, then pass the formatted messages:

```python
formatted = prompt.invoke({"role": "analyst"})
agent.invoke({"messages": formatted.messages})
```

Or just pass the system prompt when creating the agent.

### "What is `hub.pull()` and do I need it?"

`hub.pull()` downloads a pre-made prompt template from LangChain's online hub. It's totally optional — a convenience, not a requirement. You can write your own prompts instead.

### "What's `events["messages"][-1]`?"

`-1` means the last item in a list. So this grabs the most recent message from the conversation — usually the AI's latest response.

---

## Memory / Conversations

### "This conversation history approach seems static"

Wrap it in a `while True` loop to make it dynamic. Each iteration adds to `chat_history`, and the LLM sees the full conversation each time.

### "How does the chain know which branch it took earlier?"

It doesn't. LCEL chains are stateless. You need to track that yourself outside the chain, or use LangGraph for stateful workflows.

### "Where would I actually need LangGraph over simple loops?"

When you have: auto-routing after edits, AI self-evaluation loops, multiple agents collaborating, or tool calling with dynamic decisions. If data flows in one direction → LCEL + while loop. If the AI decides where to go next → LangGraph.

---

## LangChain Versions

The latest versions as of early 2026:

| Package | Version |
|---|---|
| `langchain-core` | 1.2.x |
| `langchain` | 1.2.x |
| `langchain-openai` | 1.1.x |
| `langchain-community` | 0.4.x |

All the LCEL patterns, ChatPromptTemplate, StrOutputParser, and agent creation work in these versions. The `create_agent` function was introduced in LangChain 1.0.

---

Previous: [[Packages-and-Classes]] | Back to: [[00-Index]]
