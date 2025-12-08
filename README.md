# HeartCentrix BCG LegalOps Agent – Frontend (link redirect to other GIT with Lovable repo) + Backend (n8n + Supabase)

This repository contains the backend for the **HeartCentrix × BCG AI LegalOps Assistant**.

**URL**: https://bcg-heartcentrix-poc.lovable.app 

It is implemented as a set of **n8n workflows** orchestrating:

- OpenAI (chat, embeddings, audio transcription)
- Supabase (Postgres + vector store)
- Google Drive (source PDFs)
- HTTP webhooks consumed by the React frontend

> The goal of this backend is to provide a robust, auditable and extensible LegalOps assistant for in-house legal, compliance and enterprise legal operations teams.

---

## High-Level Architecture

```text
+---------------------------+         +----------------------+
|  React Frontend (Lovable) | <-----> |  n8n Webhook (Chat)  |
+---------------------------+   POST  +----------+-----------+
                                                |
                                                v
                                        +-------+--------+
                                        |   AI Agent     |
                                        |  (OpenAI LLM)  |
                                        +---+-------+----+
                                            |       |
                                 tools: RAG |       | Memory
                                            |       |
                                            v       v
                                   +--------+-------+-----------------+
                                   |  Supabase Vector Store (RAG DB)  |
                                   +----------------------------------+

      +------------------------ n8n Scheduled Workflow ----------------------+
      |                                                                      |
      |   HTTP URLs + Google Drive PDFs  -->  Cleaning + Embeddings  --> DB |
      +---------------------------------------------------------------------+
