# HeartCentrix LegalOps – Chatbot Agent (n8n Workflow)

This workflow implements the **conversational backend** for the HeartCentrix LegalOps AI Assistant.  
It is responsible for receiving chat messages from the frontend (Lovable), handling optional
attachments (text files or voice notes), keeping conversational memory per user, and generating a
final answer using an OpenAI-based AI Agent.

---

## High-Level Flow

1. **Webhook** receives the request from the frontend.
2. The workflow checks if there is any **binary attachment**.
3. If there is an attachment:
   - If it is a **text file**, it is parsed and injected into the user prompt.
   - If it is an **audio recording**, it is transcribed to text.
4. A **Simple Memory** node loads the last turns for that user.
5. The **AI Agent** node generates a response, using:
   - The user message
   - Optional file contents / transcription
   - Conversation memory
   - A HeartCentrix-specific system prompt
6. The answer is returned to the frontend via **Respond to Webhook**.

---

## Node-by-Node Architecture

### 1. Webhook – `Webhook`

- **Role:** Entry point for all chat interactions.
- **Method:** `POST`
- **Payload types:**
  - `application/json` for pure text messages:
    ```json
    {
      "message": "User question...",
      "userId": "demo-user-001"
    }
    ```
  - `multipart/form-data` for messages with attachment:
    - `file` → binary file (audio or text)
    - `message` → optional typed text
    - `userId` → session identifier

- **Binary field name:** `file` / mapped in the workflow as `binary.file0`.

This node is also used as the **session source** (via `userId`) for the memory node.

---

### 2. Attachment Detection – `If` + `Switch`

#### `If` – “Has Attachment?”

- **Condition:** checks if `{{$binary.file0}}` is not empty.
- **True branch:** request contains a file → routed to `Switch`.
- **False branch:** no file → AI Agent will only use text from the webhook (and memory).

#### `Switch` – “Attachment Type”

Executed only when the `If` condition is true.

- **Routing key:** `{{$binary.file0.mimeType}}`
- **Cases:**
  - `audio/webm` → route to **Transcribe a recording**
  - `text/plain` → route to **Extract from File**
- **Default:** If MIME type is unknown, the file is ignored and only the text message is used.

---

### 3. Text File Handling – `Extract from File`

- **Node:** `Extract from File`
- **Operation:** `Extract From Text File`
- **Input Binary Field:** `file0`
- **Destination Output Field:** `file0`
- **Result:** The node outputs a cleaned text version of the uploaded `.txt` file
  (available under `{{ $json.data }}` in downstream nodes).

This allows the AI Agent to reason over user-provided context such as policies, notes or playbooks.

---

### 4. Audio Handling – `Transcribe a recording`

- **Node:** `Transcribe a recording`
- **Provider:** OpenAI Speech / Whisper (configured through n8n credentials).
- **Input:** `binary.file0` (audio file, typically `audio/webm` recorded from the frontend).
- **Output:** 
  - `{{ $json.text }}` → transcription of the audio
  - `{{ $json.usage.duration.seconds }}` → recording duration (used only for logging/UI).

The transcribed text is then used as the user message for the AI Agent.

---

### 5. Conversation Memory – `Simple Memory`

- **Node:** `Simple Memory`
- **Key (session id):**
  ```n8n
  {{ $json.body.userId || "default-session" }}


## High-Level Architecture

```text
+-----------------------------+        +----------------------------+
|        Lovable Frontend     |        |      n8n Cloud Instance    |
|                             |        |                            |
|  - Chat UI (messages)       |  HTTP  |  [1] Webhook               |
|  - File upload (txt/pdf)    +------->+  - Receives JSON or        |
|  - Voice recording (audio)  |        |    multipart/form-data     |
+-------------+---------------+        +-------------+--------------+
                                                  |
                                                  v
                                        +---------+-----------+
                                        |  Attachment Routing |
                                        |  - If (has file?)   |
                                        |  - Switch (type)    |
                                        +----+------------+---+
                                             |            |
                           text/plain (.txt) |            | audio/webm
                                             v            v
                                  +----------+--+   +-----+--------------+
                                  | Extract     |   | Transcribe         |
                                  | from File   |   | a Recording (STT)  |
                                  +------+------ +   +-----+-------------+
                                         \              /
                                          \            /
                                           v          v
                                         +-------------+
                                         |  Simple     |
                                         |  Memory     |
                                         +------+------+ 
                                                |
                                                v
                                        +-------+---------+
                                        |   AI Agent      |
                                        |  - System msg   |
                                        |  - Memory       |
                                        |  - User input   |
                                        +-------+---------+
                                                |
                                                v
                                      +---------+----------+
                                      | Respond to Webhook |
                                      |  - { reply: ... }  |
                                      +--------------------+
