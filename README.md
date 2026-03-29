# 🤖 RAG Chatbot Agent

An end-to-end **Retrieval-Augmented Generation (RAG)** system built with **n8n** that transforms a Google Drive folder into a searchable AI-powered knowledge base. Ask questions in plain English and get accurate, document-grounded answers.

---

## 🧠 What Is RAG?

RAG stands for **Retrieval-Augmented Generation**. Instead of relying on an AI's general training data, RAG:

1. **Retrieves** the most relevant chunks from your own documents
2. **Augments** the user's question with that retrieved context
3. **Generates** a precise answer grounded in your actual files

This means the chatbot answers from **your documents**, not from general AI knowledge.

---

## 🏗️ Architecture

```
Google Drive Folder
        ↓
  n8n Ingestion Workflow
        ↓
  Extract Text → AI Metadata Extraction → Chunking → Embeddings
        ↓
  Qdrant Vector Database
        ↓
  User Question → Embed Query → Retrieve Top Chunks
        ↓
  Groq LLM (Llama 3.3 70B) → Grounded Answer
        ↓
  Chat Interface + Google Sheets Log
```

---

## ⚙️ Tech Stack

| Layer | Tool |
|---|---|
| Workflow Orchestration | [n8n](https://n8n.io) |
| Document Storage | Google Drive |
| Vector Database | [Qdrant](https://qdrant.tech) |
| LLM — Generation & Metadata | [Groq](https://console.groq.com) — Llama 3.3 70B |
| Embeddings | Hugging Face — `sentence-transformers/all-MiniLM-L6-v2` |
| Logging | Google Sheets |
| Notifications | Gmail |

---

## 🔄 Workflow 1 — Document Ingestion Pipeline

Triggered manually or on schedule. Processes documents from Google Drive and stores them in Qdrant.

**Flow:**
```
Manual Trigger
→ Config (folder ID, collection name, chunk settings)
→ Google Drive — List PDF files
→ Filter PDFs only
→ Loop Over Files (batch size: 1)
  → Download File
  → Extract Text from PDF
  → Normalize & Clean Text
  → Groq — Extract Metadata (title, summary, keywords, topics, risks)
  → Merge text + metadata
  → Flatten Metadata
  → Chunk Text (1200 tokens, 200 overlap)
  → Generate Embeddings (HuggingFace)
  → Store in Qdrant with metadata payload
  → Log to Google Sheets
→ Gmail — Send completion notification
```

**Metadata extracted per document:**
- Title
- Summary
- Main topics
- Keywords
- Document type
- Audience
- Important entities
- Action items
- Risks
- Dates

---

## 💬 Workflow 2 — RAG Chat Interface

Runs every time a user sends a message via n8n's built-in chat UI.

**Flow:**
```
Chat Trigger
→ Master Agent (LangChain)
  → Simple Memory (last 30 messages)
  → Qdrant Vector Store Tool (Top-K: 8 chunks)
  → Groq Llama 3.3 70B
→ Return grounded answer
→ Log to Google Sheets
```

**System behaviour:**
- Always searches Qdrant before answering
- Mentions source file names in responses
- Says "I don't know" if answer is not in documents
- Maintains conversation memory across turns

---

## 🗂️ Repository Structure

```
rag_chatbot_agent/
├── README.md                        ← Project documentation
├── .gitignore                       ← Ignores secrets and env files
├── .env.example                     ← Required credentials template
└── workflows/
    └── RAG-CHATBOT-AGENT.json       ← n8n workflow (import this)
```

---

## 🚀 Getting Started

### 1. Prerequisites
- n8n instance (cloud or self-hosted)
- Qdrant cluster ([free tier available](https://cloud.qdrant.io))
- Groq API key ([free tier](https://console.groq.com))
- Hugging Face API key ([free](https://huggingface.co/settings/tokens))
- Google account (Drive + Sheets + Gmail)

### 2. Create a Qdrant Collection
In your Qdrant dashboard, create a collection with:
- **Vector size:** 384
- **Distance:** Cosine

### 3. Set Up Credentials in n8n
Go to **n8n → Settings → Credentials** and add:

| Credential | Used For |
|---|---|
| Google Drive OAuth2 | Reading files |
| Google Sheets OAuth2 | Logging |
| Gmail OAuth2 | Notifications |
| Qdrant API | Vector storage |
| Groq API | LLM generation |
| Hugging Face API | Embeddings |

### 4. Import the Workflow
1. Open your n8n instance
2. Click **New Workflow → ⋮ Menu → Import from file**
3. Select `workflows/RAG-CHATBOT-AGENT.json`

### 5. Configure the Config Node
Open the **Edit Fields** node and update:

```
folder_id         → Your Google Drive folder ID
qdrant_collection → Your Qdrant collection name
qdrant_url        → Your Qdrant cluster URL
```

### 6. Run Ingestion
1. Add PDF files to your Google Drive folder
2. Click **Execute Workflow** on the ingestion workflow
3. Wait for the Gmail completion notification
4. Verify vectors appear in your Qdrant dashboard

### 7. Start Chatting
1. Open the **Chat** trigger in n8n
2. Click the chat icon to open the chat UI
3. Ask questions about your documents

---

## 🔑 Environment Variables

Copy `.env.example` to `.env` and fill in your values:

```env
QDRANT_URL=YOUR_QDRANT_CLUSTER_URL
QDRANT_API_KEY=your_qdrant_api_key_here
GROQ_API_KEY=your_groq_api_key_here
HF_API_KEY=your_huggingface_api_key_here
GOOGLE_DRIVE_FOLDER_ID=your_google_drive_folder_id
QDRANT_COLLECTION=your_qdrant_collection_name
GOOGLE_SHEETS_ID=your_google_sheets_id
GMAIL_ADDRESS=your_gmail_address
N8N_INSTANCE_ID=your_n8n_instance_id
WEBHOOK_ID=your_webhook_id
```

---

## 📊 Google Sheets Logging

The workflow automatically logs every indexed document to Google Sheets with:

| Column | Description |
|---|---|
| timestamp | When the file was indexed |
| file_id | Google Drive file ID |
| file_name | Name of the document |
| status | `indexed` |
| collection | Qdrant collection used |
| metadata | Extracted AI metadata |
| pageContent | Chunk text stored |

---

## 💡 Key Concepts Demonstrated

- **RAG Architecture** — retrieval-grounded answer generation
- **Vector Database Design** — Qdrant with rich metadata payloads
- **Semantic Chunking** — 1200 token chunks with 200 token overlap
- **AI Metadata Extraction** — structured enrichment using Groq
- **LangChain Agent** — tool-using agent with memory
- **Workflow Automation** — end-to-end orchestration in n8n
- **Google Workspace Integration** — Drive, Sheets, Gmail APIs

---

## 📄 License

MIT License — free to use, modify, and distribute.

---

## 👤 Author

**MAHADEVAN-007**
GitHub: [@MAHADEVAN-007](https://github.com/MAHADEVAN-007)
