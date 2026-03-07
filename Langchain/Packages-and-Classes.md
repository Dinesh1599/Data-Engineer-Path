# Packages and Classes — Complete Reference

Every package, class, and function used in these notes, explained simply.

---

## Core Packages to Install

```bash
pip install langchain langchain-core langchain-openai langchain-community
pip install python-dotenv pydantic
```

---

## LLM Setup

### `langchain_openai.ChatOpenAI`

Creates an OpenAI LLM instance. Provider-specific.

```python
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(model="gpt-5-mini", temperature=0)
```

### `langchain.chat_models.init_chat_model`

Creates an LLM instance that auto-detects the provider. Newer, more universal.

```python
from langchain.chat_models import init_chat_model
llm = init_chat_model(model="gpt-5-mini", temperature=0.7)
```

---

## Messages

### `langchain_core.messages.HumanMessage`

A message from the user.

```python
from langchain_core.messages import HumanMessage
HumanMessage(content="Tell me a joke")
```

### `langchain_core.messages.SystemMessage`

Sets the behavior/personality for the LLM.

```python
from langchain_core.messages import SystemMessage
SystemMessage(content="You are a comedian")
```

### `langchain_core.messages.AIMessage`

A message from the AI. You rarely create these manually — the LLM returns them.

### `langchain_core.messages.ToolMessage`

Wraps the output of a tool call. Used when manually handling tool calls.

```python
from langchain_core.messages import ToolMessage
ToolMessage(content="search results here", tool_call_id="call_abc123")
```

---

## Prompts

### `langchain_core.prompts.PromptTemplate`

Single text template, no roles. For simple prompts.

```python
from langchain_core.prompts import PromptTemplate
prompt = PromptTemplate.from_template("Write about {topic}")
```

### `langchain_core.prompts.ChatPromptTemplate`

Multi-message template with roles. For prompts that need system messages.

```python
from langchain_core.prompts import ChatPromptTemplate
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are {role}"),
    ("human", "{question}")
])
```

**Key methods:**
- `.from_messages([...])` — create from list of tuples
- `.from_template("...")` — create a single human message
- `.invoke({...})` — fill in variables, returns PromptValue

### `langchain_core.prompts.MessagesPlaceholder`

A slot in a ChatPromptTemplate where you insert conversation history.

```python
from langchain_core.prompts import MessagesPlaceholder
MessagesPlaceholder("chat_history")
```

---

## Parsers

### `langchain_core.output_parsers.StrOutputParser`

Extracts the `.content` string from an AIMessage. Replacement for writing `.content` manually.

```python
from langchain_core.output_parsers import StrOutputParser
chain = prompt | llm | StrOutputParser()
```

**Don't use with `with_structured_output()`** — structured output already returns a parsed object.

---

## Runnables

### `langchain_core.runnables.RunnableLambda`

Wraps a regular Python function so it can be used in LCEL chains.

```python
from langchain_core.runnables import RunnableLambda

def my_func(text):
    return text.upper()

runnable = RunnableLambda(my_func)
```

### `langchain_core.runnables.RunnableSequence`

The class behind the `|` pipe operator. You never use this directly.

```python
from langchain_core.runnables import RunnableSequence
# These are identical:
chain = prompt | llm | parser
chain = RunnableSequence(prompt, llm, parser)
```

### `langchain_core.runnables.RunnableParallel`

Runs multiple chains at the same time.

```python
from langchain_core.runnables import RunnableParallel
RunnableParallel(branches={"a": chain_a, "b": chain_b})
```

### `langchain_core.runnables.RunnableBranch`

Routes to different chains based on conditions (if/else for chains).

```python
from langchain_core.runnables import RunnableBranch
RunnableBranch(
    (lambda x: "yes" in x, chain_a),  # if condition → chain_a
    chain_b                              # else → chain_b
)
```

---

## Structured Output (Pydantic)

### `pydantic.BaseModel`

Base class for defining output schemas.

```python
from pydantic import BaseModel
class MySchema(BaseModel):
    setup: str
    punchline: str
```

### `pydantic.Field`

Adds descriptions to schema fields (helps the LLM understand what to put where).

```python
from pydantic import Field
setup: str = Field(description="The setup for the joke")
```

### `.with_structured_output()`

Binds a Pydantic schema to the LLM so it returns structured data.

```python
llm_structured = llm.with_structured_output(MySchema)
result = llm_structured.invoke("Tell me a joke")
result.setup       # access with dot notation
result.model_dump() # convert to dict
```

### `typing.Literal`

Restricts output to specific values.

```python
from typing import Literal
sentiment: Literal["Positive", "Negative"]
```

### `typing.TypedDict`

Lighter alternative to Pydantic. Returns a plain dict, no runtime validation.

```python
from typing import TypedDict
class MySchema(TypedDict):
    setup: str
    punchline: str
```

---

## Tools

### `langchain_core.tools.tool`

Decorator to create custom tools. **Docstring is required.**

```python
from langchain_core.tools import tool

@tool
def my_tool(query: str) -> str:
    """Description of what this tool does — LLM reads this."""
    return "result"
```

### `langchain_community.tools.DuckDuckGoSearchRun`

Free web search tool. No API key needed.

```python
from langchain_community.tools import DuckDuckGoSearchRun
search_tool = DuckDuckGoSearchRun()
```

### `langchain_community.tools.WikipediaQueryRun`

Wikipedia search tool.

```python
from langchain_community.tools import WikipediaQueryRun
from langchain_community.utilities import WikipediaAPIWrapper
wiki_tool = WikipediaQueryRun(api_wrapper=WikipediaAPIWrapper())
```

### `.bind_tools()`

Tells the LLM which tools exist. Does NOT execute them.

```python
llm_binded = llm.bind_tools(toolkit)
```

---

## Agents

### `langchain.agents.create_agent`

Creates a ReAct agent that automatically loops through tool calls.

```python
from langchain.agents import create_agent
agent = create_agent(llm, tools)
```

Invocation:
```python
agent.invoke({"messages": [("user", "your question")]})
agent.stream({"messages": [...]}, stream_mode="values")
```

### `langchain_community.agent_toolkits.sql.toolkit.SQLDatabaseToolkit`

Pre-built toolkit for SQL databases with 4 tools (list tables, get schema, run query, check query).

```python
from langchain_community.agent_toolkits.sql.toolkit import SQLDatabaseToolkit
toolkit = SQLDatabaseToolkit(db=sql_db, llm=llm)
```

### `langchain_community.utilities.sql_database.SQLDatabase`

Connects to a SQL database.

```python
from langchain_community.utilities.sql_database import SQLDatabase
db = SQLDatabase.from_uri("sqlite:///path/to/db.db")
```

---

## Environment

### `dotenv.load_dotenv`

Loads variables from `.env` file into environment.

```python
from dotenv import load_dotenv
load_dotenv()
```

---

Previous: [[12-Memory-and-Conversations]] | Next: [[Tips-and-Common-Doubts]]
