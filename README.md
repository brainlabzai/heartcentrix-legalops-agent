# HeartCentrix LegalOps AI Agent

Este repositório reúne **frontend** (Lovable) e **backend** (n8n + Supabase) do agente:

> _HeartCentrix LegalOps AI Assistant — um chatbot focado em Legal Operations, compliance e gestão de times jurídicos corporativos._

## Arquitetura

- **frontend/** – interface de chat feita no **Lovable** (React), com:
  - Input de texto
  - Upload de arquivos `.txt` (RAG pontual)
  - Envio de áudio (transcrição automática)
- **n8n/** – workflows de automação:
  - `rag-agent.json` – agente RAG conectado ao Supabase Vector Store
  - `chat-agent.json` – fluxo de conversação e memória com o usuário
- **Supabase** – Vector Store com função `match_documents` para recuperar contexto.
- **OpenAI** – usado tanto para:
  - Embeddings (indexação e busca de contexto)
  - Respostas do agente (modelo de chat)
  - Transcrição de áudio (node “Transcribe a recording”)

## Fluxo resumido

1. Usuário interage no frontend (Lovable):
   - mensagem de texto **ou**
   - anexa `.txt` **ou**
   - envia áudio.
2. O frontend chama o **webhook do n8n**.
3. O workflow decide:
   - Se tem arquivo ⇒ extrai texto (`Extract from File`).
   - Se tem áudio ⇒ transcreve (`Transcribe a recording`).
   - Caso contrário ⇒ usa só o texto digitado.
4. O **AI Agent** no n8n:
   - Usa o vetor no Supabase (`match_documents`) para buscar contexto.
   - Aplica o **System Prompt LegalOps**.
   - Gera a resposta final.
5. A resposta volta para o frontend e aparece no chat.

## Pastas

- `frontend/` – código e assets da interface Lovable.
- `n8n/` – JSONs dos workflows + notas de configuração.
- `docs/` (opcional) – diagramas, prints e documentação complementar.

> **Status atual:** frontend ainda está versionado no repositório `heart-centrix-mockup`. Este repo será a base única (monorepo); a migração completa do frontend para `frontend/` será feita em breve.
