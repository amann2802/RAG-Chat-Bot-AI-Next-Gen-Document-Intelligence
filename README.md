# n8n RAG Chatbot Workflow

An n8n workflow that implements a Retrieval-Augmented Generation (RAG) chatbot. It has two connected pipelines: a **document ingestion pipeline** that turns uploaded documents into searchable vector embeddings, and a **chat pipeline** that lets an AI Agent answer user questions using those embeddings as context.

## Workflow Overview

### 1. Document Ingestion Pipeline
Triggered whenever a document is submitted via a form.

**Flow:** `On form submission` → `Simple Vector Store` (insert mode) → `Default Data Loader` → `Embeddings Google Gemini`

- **On form submission** — Trigger node that starts the flow when a user submits a form (e.g. uploading a document).
- **Simple Vector Store** — Receives the submitted document and prepares it for storage.
- **Default Data Loader** — Loads/parses the document content, splitting it into chunks.
- **Embeddings Google Gemini** — Converts the document chunks into vector embeddings using Google's Gemini embedding model, which are then written into the vector store.

### 2. Chat / Retrieval Pipeline
Triggered whenever a user sends a chat message.

**Flow:** `When chat message received` → `AI Agent` (uses `Google Gemini Chat Model` + `Simple Vector Store1` as a tool)

- **When chat message received** — Trigger node that starts the flow on an incoming chat message.
- **AI Agent** — The core conversational agent that processes the user's message.
  - **Chat Model:** `Google Gemini Chat Model` — generates the agent's responses.
  - **Tool:** `Simple Vector Store1` — a retriever tool the agent can call to search the vector store for relevant document chunks (using the same `Embeddings Google Gemini` model for query embedding), enabling context-aware, document-grounded answers.
  - **Memory:** available but not currently connected.

## Requirements
- n8n instance with the following nodes/credentials configured:
  - Form Trigger
  - Chat Trigger
  - AI Agent node
  - Google Gemini Chat Model credentials
  - Google Gemini Embeddings credentials
  - Vector Store node (Simple Vector Store)

## Usage
1. Submit a document through the form to ingest and embed it into the vector store.
2. Open the chat and ask questions — the AI Agent will retrieve relevant chunks from the vector store and use the Gemini chat model to generate grounded answers.

## Notes
- Both vector store nodes (`Simple Vector Store` for ingestion and `Simple Vector Store1` for retrieval) share the same underlying `Embeddings Google Gemini` model to ensure query and document embeddings are compatible.
- Enable **Memory** on the AI Agent if you want the chatbot to retain conversation context across turns.
