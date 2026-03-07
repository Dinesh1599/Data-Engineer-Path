# 12 — Memory and Conversations

## The Problem

Every `.invoke()` call is stateless — the LLM forgets everything from previous calls. To have a conversation, you need to manually send the full chat history each time.

## MessagesPlaceholder

A slot in your prompt template where previous messages get inserted:

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant"),
    MessagesPlaceholder("chat_history"),
    ("human", "{input}")
])

chain = prompt | llm | StrOutputParser()
```

## Building a Conversation Loop

```python
chat_history = []

while True:
    user_input = input("You: ")
    if user_input.lower() == "quit":
        break
    
    response = chain.invoke({
        "chat_history": chat_history,
        "input": user_input
    })
    
    # Add both messages to history
    chat_history.append(("human", user_input))
    chat_history.append(("ai", response))
    
    print(f"AI: {response}")
```

The `chat_history` list grows with each exchange. The LLM sees the full conversation every time.

## Follow-Up on Generated Content

If you generated a post using a chain and want to edit it:

```python
# Step 1: Generate the initial post
result = final_chain.invoke(invoker)

# Step 2: Create a follow-up chain
followup_prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful post editor"),
    MessagesPlaceholder("chat_history"),
    ("human", "{input}")
])
followup_chain = followup_prompt | llm | StrOutputParser()

# Step 3: Pass previous output as history
chat_history = [("human", "Generate a post"), ("ai", result)]

updated = followup_chain.invoke({
    "chat_history": chat_history,
    "input": "Add a real life example"
})
```

## Key Points

- LCEL chains are **stateless** — they don't remember anything between invocations
- **You** manage the history by maintaining a list and passing it each time
- `MessagesPlaceholder` is the slot where history gets injected
- History is just a list of `("human", "...")` and `("ai", "...")` tuples

## When Do You Need LangGraph?

For simple conversation loops, manual history management works fine. You need LangGraph when:

- The AI decides which chain to route to dynamically
- You have auto-evaluation loops (generate → check → retry)
- Multiple agents collaborate
- You need tool calling with decision-making in the loop

LangGraph doesn't do anything magical — it organizes the same logic as a graph instead of nested if/else/while loops. It's about clean structure at scale.

---

Previous: [[11-SQL-Database-Agent]] | Next: [[Packages-and-Classes]]
