# n8n Workflows

Aqui ficam os workflows relacionados ao agente **HeartCentrix LegalOps**.

## Workflows principais

- `workflows/rag-agent.json`  
  Workflow com:
  - Webhook de entrada (chat / arquivo / áudio)
  - Normalização de input (If, Extract from File, Transcribe recording)
  - `AI Agent` usando:
    - `OpenAI Chat Model`
    - `Supabase Vector Store` (função `match_documents`)
    - `Simple Memory` para contexto curto de conversa
  - `Respond to Webhook` com `reply` em JSON

- `workflows/chat-agent.json`  
  (reservado para variações futuras de agente, se necessário)

## Como exportar do n8n

1. No n8n, abra o workflow.
2. Clique em **Download → Download Workflow** (`.json`).
3. No GitHub, suba o arquivo em `n8n/workflows/` usando **Upload files**.
