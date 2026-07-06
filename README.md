# More3zdenAI

More3zdenAI is a RAG-powered portfolio assistant that lets visitors ask natural-language questions about a person's skills, projects, and experience. The app combines a FastAPI backend, a Next.js frontend, FAISS-based retrieval, and a local Ollama model to deliver grounded answers with source attribution.

It is designed as a modern, containerized AI product with streaming responses, persistent conversations, and optional voice output.

---

## Overview

More3zdenAI replaces static portfolio navigation with an interactive chat experience. Users can ask questions such as:

- What technologies do you work with?
- What projects have you built?
- What experience do you have in backend development?

The backend retrieves relevant chunks from a markdown knowledge base, generates an answer with a local LLM, and returns the results to the frontend with source references.

---

## Architecture

```text
User → Next.js UI → FastAPI API → RAG Pipeline
                              ├─ FAISS vector search
                              ├─ PostgreSQL conversation history
                              ├─ Redis response cache
                              └─ Ollama LLM generation
```

### Core components

- Frontend: Next.js 14, React, TypeScript, Tailwind CSS
- Backend: Python, FastAPI, SQLAlchemy, Pydantic
- AI / RAG: FAISS, sentence-transformers, Ollama, local embeddings
- Data: PostgreSQL for conversations, Redis for caching
- Infra: Docker Compose, Nginx, optional TTS service

---

## Features

- Conversational RAG over a portfolio knowledge base
- Grounded answers with source attribution
- Streaming chat responses using Server-Sent Events
- Persistent chat sessions in PostgreSQL
- Redis-based caching for repeat queries
- Optional text-to-speech using LuxTTS with a local fallback
- Health endpoint for service monitoring
- Docker-based deployment for local and production-friendly setups

---

## Project Structure

```text
.
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── config.py
│   │   ├── database.py
│   │   ├── models.py
│   │   ├── schemas.py
│   │   ├── cache.py
│   │   ├── rag/
│   │   └── routes/
│   ├── requirements.txt
│   └── Dockerfile
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   ├── components/
│   │   └── lib/
│   ├── package.json
│   └── Dockerfile
├── knowledge_base/
│   └── portfolio.md
├── tts/
├── docker/
├── scripts/
├── docker-compose.yml
└── .env.example
```

---

## Getting Started

### Prerequisites

- Docker Desktop or Docker Engine with Docker Compose
- Git
- At least 4 GB of free disk space for the model and dependencies

### 1. Clone the repository

```bash
git clone https://github.com/3zden/More3zdenAi.git
cd More3zdenAi
```

### 2. Configure environment

```bash
cp .env.example .env
```

Adjust the values in `.env` if needed. The defaults are suitable for local development.

### 3. Start the stack

```bash
./scripts/start.sh
```

Or run the services directly:

```bash
docker compose up -d --build
```

### 4. Open the app

- UI: http://localhost
- API docs: http://localhost:8000/docs
- Health check: http://localhost/api/health/

---

## Running the Services

### Backend logs

```bash
docker compose logs -f backend
```

### Frontend logs

```bash
docker compose logs -f frontend
```

### Stop everything

```bash
docker compose down
```

### Full reset

```bash
docker compose down -v
```

---

## Updating the Knowledge Base

Edit [knowledge_base/portfolio.md](knowledge_base/portfolio.md), then rebuild the FAISS index and restart the backend:

```bash
docker compose exec backend python -c "import os; os.remove('/app/data/faiss.index'); os.remove('/app/data/chunks.pkl')"
docker compose restart backend
```

The backend will rebuild the index on startup if the cached files are missing.

---

## API Endpoints

### Chat

- POST /api/chat/
  - Non-streaming chat request with persisted conversation history and caching
- GET /api/chat/stream/
  - Server-Sent Events endpoint for streaming responses

### Conversation

- GET /api/conversation/{session_id}/
  - Retrieve the full conversation history for a session

### Health

- GET /api/health/
  - Returns backend and service health status

### TTS

- POST /api/tts/
  - Synthesizes speech from text using the optional LuxTTS service

---

## Environment Variables

The app uses environment variables from `.env`. Key variables include:

| Variable | Purpose |
|---|---|
| POSTGRES_DB / POSTGRES_USER / POSTGRES_PASSWORD | Database configuration |
| REDIS_URL | Redis connection string |
| OLLAMA_BASE_URL / OLLAMA_MODEL | Ollama LLM settings |
| EMBEDDING_MODEL | Sentence-transformer embedding model |
| RAG_TOP_K / RAG_MIN_SCORE | Retrieval settings |
| TTS_ENABLED / TTS_BASE_URL | Optional voice output settings |
| NEXT_PUBLIC_API_URL | Frontend API base URL |

---

## Development Notes

- The backend expects its supporting services to be reachable via the Docker network hostnames such as `db`, `redis`, and `ollama`.
- The FAISS index is built on first startup and persisted in the `faiss_data` volume.
- The TTS service is optional and can be enabled with the `voice` profile in Docker Compose.

---

## License

This project is intended for personal portfolio and demonstration use.
