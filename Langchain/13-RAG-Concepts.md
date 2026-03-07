# 13 — RAG Concepts (High-Level Only)

> **No code in this file.** This is the conceptual overview covered at the end of the course. RAG implementation (embeddings, vector stores, document loaders, retrievers) is a separate deep topic.

## The Problem

LLMs are not trained on your data. They can't access your company's database, your PDFs, your internal docs. If you ask "How much sales did we make last quarter?" — the LLM has no idea.

## The Solution: RAG

**RAG = Retrieval Augmented Generation**

Break it into three words:

| Step | What it means |
|---|---|
| **Retrieval** | Find the relevant data from your files/database |
| **Augmented** | Enrich the LLM's knowledge with that retrieved data |
| **Generation** | LLM generates an answer using the retrieved context |

## How It Works (Step by Step)

### 1. Your data gets converted to vectors

Your documents (PDFs, CSVs, text files, database records) get split into chunks and converted into **vectors** — lists of numbers that represent the meaning of the text.

```
"Total sales for Q3 were $2M" → [0.12, 0.85, 0.33, 0.67, ...]
"Marketing budget was $500K"  → [0.45, 0.21, 0.78, 0.11, ...]
"New hires in engineering: 15" → [0.91, 0.04, 0.56, 0.38, ...]
```

These vectors get stored in a **vector database** (like Pinecone, Chroma, FAISS).

### 2. Your question also becomes a vector

When you ask "How much sales did we make?", that question gets converted to a vector too:

```
"How much sales did we make?" → [0.14, 0.82, 0.31, 0.70, ...]
```

### 3. Similarity search finds the closest match

The system compares your question vector against all stored vectors using **cosine similarity** (or Euclidean distance). It finds the most similar one:

```
Question vector: [0.14, 0.82, 0.31, 0.70, ...]
Closest match:   [0.12, 0.85, 0.33, 0.67, ...]  ← "Total sales for Q3 were $2M"
```

### 4. The matched text goes to the LLM as context

Now the LLM receives both your question AND the retrieved text:

```
Context: "Total sales for Q3 were $2M"
Question: "How much sales did we make?"
→ LLM can now answer: "Total sales for Q3 were $2 million."
```

Without RAG, the LLM would say "I don't have access to your sales data."

## Visual Summary
![[Visual Summary of how RAG works.png]]

## Why "Retrieval Augmented Generation"?

- **Retrieval** — we retrieved the similar vector/text from the database
- **Augmented** — we augmented (enriched) the LLM with data it didn't have before
- **Generation** — the LLM generated the final answer using that context

## What You'd Need to Learn Next (Not Covered Here)

- **Document loaders** — reading PDFs, CSVs, web pages into LangChain
- **Text splitters** — chunking large documents into smaller pieces
- **Embeddings** — the models that convert text to vectors (OpenAI, HuggingFace, etc.)
- **Vector stores** — databases for storing vectors (Pinecone, Chroma, FAISS, Weaviate)
- **Retrievers** — the search layer that finds relevant chunks
- **RAG chains** — combining retrieval with LLM generation in LCEL

All of these use the same LangChain fundamentals you already know — prompts, chains, LCEL, parsers. RAG just adds a retrieval step before the LLM call.

---

Previous: [[12-Memory-and-Conversations]] | Next: [[Packages-and-Classes]]
