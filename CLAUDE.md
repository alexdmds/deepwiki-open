# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DeepWiki-Open is an AI-powered wiki generator for GitHub, GitLab, and Bitbucket repositories. It clones repos, builds vector embeddings, generates documentation via LLMs, and renders an interactive wiki with Mermaid diagrams and RAG-powered chat.

**Stack**: Python FastAPI backend (`api/`, port 8001) + Next.js 15 frontend (`src/`, port 3000), packaged as a monorepo.

## Common Commands

### Frontend
```bash
yarn install                    # Install JS dependencies
yarn dev                        # Dev server (Turbopack, port 3000)
yarn build                      # Production build (standalone output)
yarn lint                       # ESLint
```

### Backend
```bash
poetry install -C api           # Install Python dependencies
python -m api.main              # Start backend (hot-reload in dev, port 8001)
```

### Tests
```bash
python tests/run_tests.py              # All tests
python tests/run_tests.py --unit       # Unit tests only (tests/unit/)
python tests/run_tests.py --integration # Integration tests (tests/integration/)
python tests/run_tests.py --api        # API tests (tests/api/, requires running server)
```

Note: `pytest.ini` points to `test/` (singular) with standalone tests. The main test suite lives in `tests/` (plural) and uses its own runner.

### Docker
```bash
docker-compose up                              # Full stack (frontend + backend)
docker-compose -f docker-compose.backend.yml up # Backend only
```

## Architecture

### Backend (`api/`)

- **`api/main.py`** — Entry point. Loads `.env`, configures Google GenAI, starts Uvicorn.
- **`api/api.py`** — FastAPI app with all REST endpoints (wiki cache CRUD, export, model config, auth, health).
- **`api/websocket_wiki.py`** — WebSocket handler at `/ws/chat` for wiki generation streaming.
- **`api/simple_chat.py`** — HTTP SSE fallback at `/chat/completions/stream`.
- **`api/rag.py`** — RAG implementation using FAISS retriever + conversation memory.
- **`api/data_pipeline.py`** — Repo cloning, file filtering, token counting, embedding via `DatabaseManager`.
- **`api/config.py`** — Loads JSON configs from `api/config/` (`generator.json`, `embedder.json`, `repo.json`, `lang.json`). Resolves `${ENV_VAR}` placeholders. Exposes `get_model_config(provider, model)`.
- **`api/prompts.py`** — All LLM prompt templates.
- **Provider clients** — `openai_client.py`, `openrouter_client.py`, `azureai_client.py`, `bedrock_client.py`, `dashscope_client.py`, `google_embedder_client.py`, `ollama_patch.py`.

AI framework: [AdalFlow](https://github.com/SylphAI-Inc/AdalFlow) for embeddings, RAG, and LLM abstraction.

### Frontend (`src/`)

- **App Router** — `src/app/[owner]/[repo]/page.tsx` is the main wiki viewer (dynamic route).
- **Key components** — `Ask.tsx` (chat/RAG with DeepResearch toggle), `Mermaid.tsx` (diagram renderer with pan/zoom), `Markdown.tsx` (react-markdown), `ConfigurationModal.tsx` (model/provider selection).
- **i18n** — Custom `LanguageContext` in `src/contexts/` with JSON messages in `src/messages/` (10 languages). Not using next-intl middleware routing.
- **WebSocket client** — `src/utils/websocketClient.ts` connects to `ws://localhost:8001/ws/chat`.

### API Proxying

`next.config.ts` uses `rewrites()` to proxy `/api/wiki_cache/*`, `/export/wiki/*`, `/local_repo/structure`, `/api/auth/*`, and `/api/lang/config` to the backend at `SERVER_BASE_URL`.

### Data Storage

All persistent data lives under `~/.adalflow/`:
- `repos/` — Cloned repositories
- `databases/` — FAISS vector indexes
- `wikicache/` — Cached wiki JSON, keyed as `deepwiki_cache_{type}_{owner}_{repo}_{lang}.json`

### Multi-Provider LLM

`api/config/generator.json` defines providers (google, openai, openrouter, ollama, bedrock, azure, dashscope). `get_model_config()` returns `{model_client, model_kwargs}` for any registered provider.

### Embedders

Set via `DEEPWIKI_EMBEDDER_TYPE` env var. Options: `openai` (default, `text-embedding-3-small`), `google`, `ollama`, `bedrock`. Factory in `api/tools/embedder.py`.

## Key Environment Variables

- `GOOGLE_API_KEY` / `OPENAI_API_KEY` / `OPENROUTER_API_KEY` — LLM provider keys
- `DEEPWIKI_EMBEDDER_TYPE` — Embedder selection (openai/google/ollama/bedrock)
- `SERVER_BASE_URL` — Backend URL for Next.js proxying (default `http://localhost:8001`)
- `PORT` — Backend port (default 8001)
- `DEEPWIKI_CONFIG_DIR` — Custom config directory override
- `DEEPWIKI_AUTH_MODE` / `DEEPWIKI_AUTH_CODE` — Optional auth gate
