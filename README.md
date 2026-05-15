# 🤖 ChatPDF — AI-Powered PDF Q&A Chatbot

> Upload any PDF and chat with it using natural language — powered by LangChain, LangGraph, and OpenAI.

[![MIT License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Node.js](https://img.shields.io/badge/Node.js-v18%2B-brightgreen)](https://nodejs.org/)
[![LangChain](https://img.shields.io/badge/LangChain-Enabled-blue)](https://js.langchain.com/)
[![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4-orange)](https://openai.com/)

---

## 📌 Short Description

**ChatPDF** is a full-stack AI chatbot that lets users upload PDF documents and ask questions in plain English. It extracts, embeds, and retrieves relevant content from the PDFs to generate accurate, context-aware answers in real time — using LangChain, LangGraph, Supabase (vector store), and OpenAI.

---

**Here's what the Chatbot UI looks like:**

<img width="1096" alt="ChatPDF UI Screenshot" src="https://github.com/user-attachments/assets/3a9ddea7-b718-476b-bdae-38839be20c12" />

---

## 📚 Table of Contents

1. [Features](#features)
2. [Architecture Overview](#architecture-overview)
3. [Prerequisites](#prerequisites)
4. [Installation](#installation)
5. [Environment Variables](#environment-variables)
   - [Frontend Variables](#frontend-variables)
   - [Backend Variables](#backend-variables)
6. [Local Development](#local-development)
   - [Running the Backend](#running-the-backend)
   - [Running the Frontend](#running-the-frontend)
7. [Usage](#usage)
   - [Uploading/Ingesting PDFs](#uploadingingesting-pdfs)
   - [Asking Questions](#asking-questions)
   - [Viewing Chat History](#viewing-chat-history)
8. [Production Build & Deployment](#production-build--deployment)
9. [Customizing the Agent](#customizing-the-agent)
10. [Troubleshooting](#troubleshooting)
11. [Tech Stack](#tech-stack)

---

## ✨ Features

- **📄 Document Ingestion**: Upload and parse PDFs into `Document` objects, then store vector embeddings in a Supabase vector database.
- **🔍 Intelligent Retrieval**: Handle user questions, decide whether to retrieve documents or give a direct answer, then generate concise responses with source references.
- **⚡ Streaming Responses**: Real-time streaming of partial responses from the server to the client UI.
- **🧠 LangGraph Integration**: Built using LangGraph's state machine to orchestrate ingestion and retrieval, and debug each step of the agent graph.
- **💻 Next.js Frontend**: Allows file uploads, real-time chat, and easy extension with React components and Tailwind CSS.

---

## 🏗️ Architecture Overview

```ascii
┌─────────────────────┐    1. Upload PDFs    ┌───────────────────────────┐
│Frontend (Next.js)   │ ────────────────────> │Backend (LangGraph)       │
│ - React UI w/ chat  │                      │ - Ingestion Graph         │
│ - Upload .pdf files │ <────────────────────┤   + Vector embedding via  │
└─────────────────────┘    2. Confirmation   │     SupabaseVectorStore   │
                                              └───────────────────────────┘
                                               (storing embeddings in DB)

┌─────────────────────┐    3. Ask questions  ┌───────────────────────────┐
│Frontend (Next.js)   │ ────────────────────> │Backend (LangGraph)       │
│ - Chat + SSE stream │                      │ - Retrieval Graph         │
│ - Display sources   │ <────────────────────┤   + Chat model (OpenAI)   │
└─────────────────────┘ 4. Streamed answers  └───────────────────────────┘
```

- **Supabase** — vector store to store and retrieve relevant document embeddings at query time.
- **OpenAI** (or other LLM providers) — language model for generating answers.
- **LangGraph** — orchestrates the graph steps for ingestion, routing, and response generation.
- **Next.js** (React) — powers the UI for uploading PDFs and real-time chat.

The system consists of:
- **Backend**: A Node.js/TypeScript service with LangGraph agent graphs:
  - **Ingestion** (`src/ingestion_graph.ts`) — handles indexing/ingesting documents
  - **Retrieval** (`src/retrieval_graph.ts`) — question-answering over ingested documents
  - **Configuration** (`src/shared/configuration.ts`) — model providers and vector store config
- **Frontend**: A Next.js/React app for uploading PDFs and chatting with the AI.

---

## ⚙️ Prerequisites

1. **Node.js v18+** (Node v20 recommended)
2. **Yarn** (or npm; this monorepo is configured with Yarn)
3. **Supabase project** for vector storage:
   - `SUPABASE_URL`
   - `SUPABASE_SERVICE_ROLE_KEY`
   - A table named `documents` and a function `match_documents` for vector similarity search
4. **OpenAI API Key** (or another LLM provider supported by LangChain)
5. **LangChain API Key** *(optional, but recommended for tracing/debugging)*

---

## 🚀 Installation

1. **Clone** the repository:

   ```bash
   git clone https://github.com/Tanisha10433/ChatPDF-PDF-QA-Bot.git
   cd ChatPDF-PDF-QA-Bot
   ```

2. **Install dependencies** (from the monorepo root):

   ```bash
   yarn install
   ```

3. **Configure environment variables** in both `backend/` and `frontend/`. See `.env.example` files in each folder.

---

## 🔑 Environment Variables

### Frontend Variables

```bash
cp frontend/.env.example frontend/.env
```

```env
NEXT_PUBLIC_LANGGRAPH_API_URL=http://localhost:2024
LANGCHAIN_API_KEY=your-langsmith-api-key-here        # Optional: LangSmith API key
LANGGRAPH_INGESTION_ASSISTANT_ID=ingestion_graph
LANGGRAPH_RETRIEVAL_ASSISTANT_ID=retrieval_graph
LANGCHAIN_TRACING_V2=true                            # Optional: Enable LangSmith tracing
LANGCHAIN_PROJECT="pdf-chatbot"                      # Optional: LangSmith project name
```

### Backend Variables

```bash
cp backend/.env.example backend/.env
```

```env
OPENAI_API_KEY=your-openai-api-key-here
SUPABASE_URL=your-supabase-url-here
SUPABASE_SERVICE_ROLE_KEY=your-supabase-service-role-key-here
LANGCHAIN_TRACING_V2=true                            # Optional
LANGCHAIN_PROJECT="pdf-chatbot"                      # Optional
```

**Variable Reference:**

| Variable | Description |
|---|---|
| `NEXT_PUBLIC_LANGGRAPH_API_URL` | URL of the LangGraph backend (default: `http://localhost:2024`) |
| `LANGCHAIN_API_KEY` | LangSmith API key for tracing (optional) |
| `LANGGRAPH_INGESTION_ASSISTANT_ID` | LangGraph assistant ID for ingestion (default: `ingestion_graph`) |
| `LANGGRAPH_RETRIEVAL_ASSISTANT_ID` | LangGraph assistant ID for Q&A (default: `retrieval_graph`) |
| `OPENAI_API_KEY` | Your OpenAI API key |
| `SUPABASE_URL` | Your Supabase project URL |
| `SUPABASE_SERVICE_ROLE_KEY` | Your Supabase service role key |

---

## 💻 Local Development

This monorepo uses Turborepo to manage both backend and frontend.

### Running the Backend

```bash
cd backend
yarn langgraph:dev
```

This launches a local LangGraph server on port `2024` by default.

### Running the Frontend

```bash
cd frontend
yarn dev
```

Access the UI at: **http://localhost:3000**

---

## 📖 Usage

Once both services are running:

1. Navigate to **http://localhost:3000**.
2. Click the **paperclip icon** (📎) in the chat input to upload a PDF (max 5 files, each under 10MB).
3. After ingestion completes, **type your question** in the chat input.
4. The chatbot retrieves relevant content from the PDF and streams an AI-generated answer.

### Uploading/Ingesting PDFs

- Click the paperclip icon in the chat input area.
- Select up to **5 PDF files**, each **under 10MB**.
- The backend processes the PDFs, extracts text, and stores embeddings in Supabase.

### Asking Questions

- Type your question in the chat input field.
- Responses **stream in real time**.
- If documents were retrieved, a **"View Sources"** link appears for each chunk used.

### Viewing Chat History

- A unique thread is created per user session.
- Chat history is **not persistent across sessions** by default (can be extended with a database).
- Ingested documents **are persistent** as they live in the Supabase vector database.

---

## 🌐 Deployment

### Backend

Deploy your LangGraph agent to [LangGraph Cloud](https://langchain-ai.github.io/langgraph/cloud/quick_start/) or [self-host it](https://langchain-ai.github.io/langgraph/how-tos/deploy-self-hosted/).

### Frontend

Deploy the Next.js frontend to **Vercel**, **Netlify**, or any platform that supports Next.js.

> Make sure to set `NEXT_PUBLIC_LANGGRAPH_API_URL` to point to your deployed backend URL.

---

## 🛠️ Customizing the Agent

### Backend

- Change default configs (vector store, k-value, filter kwargs) in `src/shared/configuration.ts`.
- Adjust prompts in `src/retrieval_graph/prompts.ts`.
- Add a custom retriever in `src/shared/retrieval.ts`.

### Frontend

- Modify file upload restrictions in `app/api/ingest` route.
- Change default config objects (model provider, k-value, retriever provider) in `constants/graphConfigs.ts`.

---

## 🐛 Troubleshooting

| Problem | Solution |
|---|---|
| `.env` not loaded | Copy `.env.example` to `.env` in both `backend/` and `frontend/` and restart |
| Supabase vector store errors | Ensure `documents` table and `match_documents` function are set up correctly |
| OpenAI errors | Verify `OPENAI_API_KEY` and check API quota |
| LangGraph not running | Confirm Node.js version is >= 18 and all dependencies are installed |
| Network errors | Ensure `NEXT_PUBLIC_LANGGRAPH_API_URL` points to the correct backend URL |

---

## 🧰 Tech Stack

| Layer | Technology |
|---|---|
| **Frontend** | Next.js, React, Tailwind CSS |
| **Backend** | Node.js, TypeScript, LangGraph, LangChain |
| **LLM** | OpenAI GPT-4 (configurable) |
| **Vector Store** | Supabase pgvector |
| **Orchestration** | LangGraph (state machine graphs) |
| **Monorepo** | Turborepo + Yarn Workspaces |

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).

---

*Made with ❤️ by [Tanisha](https://github.com/Tanisha10433)*
