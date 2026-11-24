# Agentic AI · Rakesh

Agentic AI applications built with **LangGraph**: planning agents, tool calling, model routing,
grounded answers, guardrails, human approval, observability and tests.

| Project | What it does | Agent & framework | Tests |
|---|---|---|---|
| [**Rayda Fleet Copilot**](https://github.com/agentic-ai-rakesh/rayda-fleet-copilot) | Natural-language copilot for IT device-fleet telemetry: grounded Q&A, trend insights, human-approved actions | LangGraph agent loop with an approval interrupt · SQLite · Claude / GPT / Gemini switch | 226 tests · 54/54 evals |
| [**Agentic RAG Pipeline Analyzer**](https://github.com/agentic-ai-rakesh/agentic-rag-pipeline-analyzer) | Build a RAG pipeline step by step and chat with your documents, with citations | LangGraph orchestrator, RAG, ingestion and fallback agents · LiteLLM auto routing · FAISS · FastAPI + Streamlit | 97 tests |
| [**News Research Agent**](https://github.com/agentic-ai-rakesh/news-research-agent) | Index news article URLs and ask questions; answers cite the articles | LangGraph research agent (plan → search tools → cited answer) · FAISS · OpenAI GPT-5.5 · FastAPI + Streamlit | 37 tests |
| [**AtliQ Tees SQL Agent**](https://github.com/agentic-ai-rakesh/atliq-tees-sql-agent) | Ask a t-shirt store's database questions in plain English; answers show the SQL and rows | LangGraph SQL agent (schema → read-only SQL → self-correction) · sqlglot guard · SQLite/MySQL · Gemini/OpenAI · FastAPI + Streamlit | 58 tests |
| [**Smart Interview**](https://github.com/agentic-ai-rakesh/smart-interview) | Interview-practice app: topic-aware questions and answers, ask anything, quick topic cards | LangGraph agent with auto model selection · OpenRouter · MongoDB · FastAPI + Streamlit | 87 tests |

---

## 🛰️ Rayda Fleet Copilot

IT admins ask questions about device telemetry (3 companies, 25 devices, 30 days) and get answers
that cite the exact snapshots. They can also act on them, with a human approving each action.

**Flow**
- Input guard → planner → tool loop (12 company-scoped tools) → responder → citation verifier
- → action proposer → evidence gate → **human approval** (LangGraph `interrupt`) → execution → audit log

**Highlights**
- Code computes, the model explains: filters, counts and trends are deterministic Python/SQL, and
  every fact carries a snapshot ID as evidence.
- Guardrails are written in code: company isolation, default-deny approval, idempotent actions, and
  a tamper-evident (hash-chained) audit log.
- Provider switch in config (Claude, GPT-5.5, Gemini). Evals score answers against ground truth
  computed from the raw data, and recorded runs make them deterministic.
- **Tech:** Python 3.11 · LangGraph · LangChain · SQLite · pydantic · Typer · pytest · uv

## 📚 Agentic RAG Pipeline Analyzer

Build and compare RAG pipelines stage by stage, then chat over them with agents that route,
retrieve, cite and recover.

**Flow**
- PDF / .docx / URL → load → chunk (5 methods) → embed (TF-IDF … sentence-transformers) → FAISS
- Question → 🧭 orchestrator agent → docs RAG / URLs RAG / ingestion agent → cache → requery
- → dense / BM25 / hybrid search → rerank → guardrails → routed LLM → answer with `[n]` citations
- No answer possible → 🛟 fallback agent explains why and offers a retry

**Highlights**
- Every parameter can be set (chunking, embeddings, index, top K, search mode, reranker), with
  recommended values measured from your document.
- Auto model routing across 8 models (OpenRouter or local Ollama), with fallback by tier.
- Guardrails: prompt injection, PII masking, blocked terms; five engines.
- Observability: LangSmith traces, the Pipeline Watch dashboard, time per step and per agent,
  tokens and cost.
- **Tech:** Python · LangGraph · LangChain · LiteLLM · FAISS · FastAPI · Streamlit · MongoDB · LangSmith

## 📰 News Research Agent (RockyBot)

Paste news URLs and ask questions; a research agent searches the articles and answers with
citations.

**Flow**
- URLs → fetch + extract article text → chunk (1000 / 200 overlap) → OpenAI embeddings → FAISS
- Question → 🤖 agent plans → `list_articles` / `search_articles` (up to 4 rounds) → answer with `[A#]`
- Not in the articles → says so, no guessing

**Highlights**
- Explicit LangGraph loop (agent ⇄ tools → finalize), one search per part of a comparison,
  capped rounds so it always ends.
- Per-URL failure reporting, atomic index rebuild, manifest that rejects a mismatched embedding
  model, no pickle.
- Every answer shows its agent steps, tokens, latency and model; optional LangSmith tracing.
- **Tech:** Python 3.11 · LangGraph · LangChain · OpenAI · FAISS · BeautifulSoup · FastAPI ·
  Streamlit · pytest · `make setup`

## 👕 AtliQ Tees SQL Agent

Store managers ask about inventory, prices and discounts; the agent queries the database and
shows its work.

**Flow**
- Question → 🤖 agent → `list_tables` / `describe_table` (exact values) → `run_sql`
- SQL error → the agent reads it and fixes the query → answer from the rows
- Write requests ("delete …") are refused: the database is read-only

**Highlights**
- Two safety layers: a sqlglot guard (one SELECT, known tables, no writes/PRAGMA/SLEEP) and a
  read-only connection with a time limit and row cap.
- Every answer shows its SQL and result rows; numbers not found in the results are flagged.
- Deterministic data (`make db`, SQLite by default, same data as a MySQL script); tests compute
  expected values in Python.
- **Tech:** Python 3.11 · LangGraph · LangChain · SQLAlchemy · sqlglot · Gemini / OpenAI · FastAPI ·
  Streamlit · pytest · `make setup`

## 🎯 Smart Interview

Interview practice by topic, subtopic, level and question type, plus an *Ask a question* tab and
*Quick Topic* explainer cards.

**Flow**
- Request → route task → select model (agent picks, rules as fallback) → generate → validate
  (one retry) → save to MongoDB
- Revisit: the *Own knowledge base* option serves saved questions with **no LLM call**

**Highlights**
- Auto model selection: the agent reads each model's description and picks Gemini Flash-Lite,
  GPT-4o mini or Claude Sonnet for the request.
- Every question and answer is stored in MongoDB, with checked/unchecked status, refine-in-simpler-
  words, and paged history.
- Per-request and per-step timings, tokens, cost and runtime metrics.
- **Tech:** Python · LangGraph · LangChain · OpenRouter · FastAPI · Streamlit · MongoDB · pytest

---

Each repository has its own README with setup steps, configuration and test instructions.
