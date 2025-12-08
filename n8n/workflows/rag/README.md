# HeartCentrix LegalOps – RAG Agent (n8n + Supabase)

This document describes the **RAG Agent** implemented in n8n for the HeartCentrix LegalOps project.

The RAG Agent is responsible for:

1. **Building and refreshing the knowledge base** used by the chatbot.
2. **Serving as a tool** that the AI Agent can call to retrieve semantically relevant documents from Supabase.

It combines:

- **n8n** – workflow orchestration
- **OpenAI** – embeddings
- **Supabase** – Postgres + pgvector
- **External sources** – curated URLs and Google Drive PDFs

---

## 1. Purpose

Traditional Q&A over a single prompt is not enough for LegalOps use cases. We need:

- Up-to-date content (guides, policies, best practices)
- Company-specific information (HeartCentrix materials, client decks)
- Structured retrieval with ranking and metadata

The RAG Agent solves this by:

- Periodically ingesting content from trusted sources
- Converting raw HTML / PDF files into clean text
- Creating vector embeddings and storing them in Supabase
- Exposing a **`match_documents`**-style semantic search function that the chatbot can call as a tool.

---

## 2. High-Level Architecture

```text
+------------------------+
|  n8n RAG Workflow(s)   |
+-----------+------------+
            |
            v
  [Fetch & Clean Content]
            |
            v
   [OpenAI Embeddings]
            |
            v
+-----------+------------+
| Supabase Vector Table  |
|  (documents + vectors) |
+-----------+------------+
            ^
            |
      [Tool Call]
            |
+-----------+------------+
| n8n Chatbot AI Agent   |
+------------------------+
