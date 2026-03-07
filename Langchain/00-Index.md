# LangChain Beginner's Guide — Index

> Personal notes from learning LangChain. Covers fundamentals through building autonomous SQL agents.
> Built while following [Anshl Lamba's LangChain Full Course](https://github.com/anshlambagit/Langchain_Tutorial) + my own explorations.

## How to Use These Notes

Read them in order. Each file links to the next. The concepts build on each other — skipping ahead will confuse you.

## Chapters

| # | Topic | What You'll Learn |
|---|-------|-------------------|
| 01 | [[01-Fundamentals]] | What is LangChain, why use it, first LLM call |
| 02 | [[02-Messages]] | System, Human, AI messages and how they work |
| 03 | [[03-Prompts-and-Templates]] | PromptTemplate vs ChatPromptTemplate, dynamic prompts |
| 04 | [[04-Structured-Output]] | Pydantic BaseModel, TypedDict, Field, guiding LLM output |
| 05 | [[05-LCEL-and-Chains]] | Pipe operator, how LCEL works, RunnableSequence |
| 06 | [[06-Custom-Runnables]] | RunnableLambda, dictionary makers, custom functions in chains |
| 07 | [[07-Parallel-Chains]] | RunnableParallel, running multiple chains at once |
| 08 | [[08-Conditional-Chains]] | RunnableBranch, routing based on conditions |
| 09 | [[09-Tools-and-Binding]] | Creating tools, bind_tools, manual tool calling flow |
| 10 | [[10-ReAct-Agents]] | What is ReAct, create_react_agent, streaming |
| 11 | [[11-SQL-Database-Agent]] | SQLDatabaseToolkit, building an autonomous DB agent |
| 12 | [[12-Memory-and-Conversations]] | MessagesPlaceholder, chat history, follow-up conversations |
| 13 | [[13-RAG-Concepts]] | What is RAG, vectors, similarity search (concepts only, no code) |

## Reference Files

| File | What's Inside |
|------|---------------|
| [[Packages-and-Classes]] | Every import, class, and function explained |
| [[Tips-and-Common-Doubts]] | FAQs, gotchas, and things that confused me |

## What's NOT Covered Here

- **RAG implementation** — only the high-level concept is covered (Chapter 13). The actual code (embeddings, vector stores, document loaders, retrievers) is a deep topic on its own.
- **LangGraph deep dive** — mentioned where relevant but not the focus of these notes.
- **IDE setup / Git setup / UV package manager** — basic dev tooling, not LangChain-specific.
- **OpenAI API key creation / billing** — standard setup, Google it if needed.

## Prerequisites

- Basic Python (functions, loops, if/else, classes, objects)
- An OpenAI API key with a few dollars loaded
- A code editor (VS Code recommended)

---

Next: [[01-Fundamentals]]
