# TEACH ME — Enterprise Knowledge Assistant, from zero

> **Who this is for:** a student who knows a little Python (variables, functions, `pip install`) and has never built an AI app before.
> By the end of this file, you will understand **what RAG is**, **why it matters**, and **how every line in the `app/` folder works**.

We will go slowly. No magic. No hand-waving.

---

## 0. The 30-second pitch

You have a folder of company PDFs — HR policies, onboarding docs, engineering wikis. You want an employee to be able to ask:

> "How many vacation days do I get in my second year?"

…and get a correct answer **with a citation**, not a hallucination.

That is what we are building. The pattern is called **RAG** — Retrieval-Augmented Generation. It is the single most common way real companies (Google, Microsoft, Amazon, every startup with a chatbot over their docs) use LLMs in production today.

---

## 1. Why not just paste the docs into ChatGPT?

Three reasons:

1. **Context windows are finite.** Even a 200k-token model can't fit your whole 50,000-page company wiki.
2. **Cost.** Sending all the docs every time someone asks a question is wildly expensive.
3. **Hallucination.** If you don't *force* the model to use specific text, it will make things up.

RAG fixes all three. Instead of giving the model *everything*, we **find the most relevant 4–8 paragraphs** for each question and give it only those.

---

## 2. The whole picture in one diagram

```
            ┌──────────────────────────── INGEST (do this once per file) ────────────────────┐
            │                                                                                │
  PDF ─►  load text  ─►  split into chunks  ─►  embed each chunk  ─►  store in FAISS index   │
            │                                                                                │
            └────────────────────────────────────────────────────────────────────────────────┘

            ┌──────────────────────────── ASK (every user question) ─────────────────────────┐
            │                                                                                │
 question ─► embed question ─► search FAISS ─► top-K chunks ─► stuff into prompt ─► LLM ─► answer
            │                                                                                │
            └────────────────────────────────────────────────────────────────────────────────┘
```

Two pipelines. Read this diagram three times. Everything that follows is just code for these arrows.

---

## 3. The four magic words

Before any code, learn these four ideas. They're the entire field.

### 3.1 Embedding
An **embedding** is a list of numbers (typically 384 or 1536 of them) that represents the *meaning* of a piece of text. Two pieces of text with similar meaning have similar numbers.

```
"How much vacation do I get?"   →   [0.21, -0.04, 0.88, ..., 0.13]
"PTO policy for new hires"      →   [0.19, -0.02, 0.85, ..., 0.11]   ← similar!
"Today's lunch menu"            →   [-0.7, 0.6, 0.01, ..., -0.4]    ← very different
```

Similarity is measured with **cosine similarity** — basically the angle between the two vectors. Close angle = similar meaning.

We use a free, local model called `all-MiniLM-L6-v2` so you don't need an API key just to embed text. It runs on your laptop's CPU.

### 3.2 Chunk
LLMs and embedding models can only handle so much text at once. So we **chop** every document into ~800-character pieces called **chunks**. Each chunk is small enough to embed cleanly and big enough to contain a complete thought.

We make chunks **overlap** by ~120 characters so that an answer that happens to sit on a chunk boundary doesn't get cut in half.

### 3.3 Vector store (FAISS)
A **vector store** is a database that stores embeddings and answers one question really fast: *"give me the K vectors closest to this one."*

**FAISS** (Facebook AI Similarity Search) is a free vector store you can run on your own machine. **Pinecone** is a hosted alternative — same idea, someone else's server. We use FAISS because it's free and local.

### 3.4 Prompt stuffing
Once we've found the top-K relevant chunks, we paste them into a prompt that looks like:

```
You are a knowledge assistant. Answer using ONLY the context below.
If it's not in the context, say "I don't know."

CONTEXT:
[chunk 1 text]
[chunk 2 text]
[chunk 3 text]

QUESTION: How many vacation days?
```

This is called **stuffing** the context. It's the boring trick that makes RAG work.

---

## 4. The codebase, file by file

The whole app is **6 small Python files**. Open each one as you read.

### 4.1 `config.py` — settings in one place
All configuration (file paths, model names, API keys, chunk size) lives here. The rest of the code imports `settings` and never touches `os.environ` directly. This is a habit — it makes testing and deployment easier.

### 4.2 `ingest.py` — turn a file into searchable chunks
This file implements the **top half** of the diagram. Read it top to bottom; the four steps are clearly labeled:

1. `load_file()` — picks the right loader based on extension (`.pdf` → `PyPDFLoader`, `.docx` → `Docx2txtLoader`, etc.) and returns LangChain `Document` objects.
2. `split_documents()` — uses `RecursiveCharacterTextSplitter` to chop the document into 800-char chunks with 120-char overlap.
3. `get_embeddings()` — loads the local sentence-transformer model. Cached so we only load it once.
4. `add_to_index()` — pushes the chunks into FAISS. If an index already exists on disk, it loads it and *adds* to it. Otherwise it creates a fresh one.

Run it manually to test:
```bash
python ingest.py path/to/handbook.pdf
# {'file': 'handbook.pdf', 'pages_or_sections': 42, 'chunks_added': 187}
```

### 4.3 `rag.py` — answer a question
This is the **bottom half** of the diagram.

1. `load_vectorstore()` — reads the FAISS index back from disk.
2. `get_llm()` — picks Claude or OpenAI based on `LLM_PROVIDER` in `.env`.
3. The **`SYSTEM_PROMPT`** is the most important piece. Read it carefully. Every word is doing work:
   - "ONLY the context below" → no hallucination
   - "reply exactly: I don't know based on the provided documents" → graceful failure
   - "Cite sources inline like [source: filename]" → traceable answers
4. `answer_question()` — runs the whole pipeline: similarity search → format context → call LLM → return both the answer and the chunks we used.

Run it manually:
```bash
python rag.py "How many vacation days?"
```

### 4.4 `main.py` — wrap it in a web API
FastAPI turns Python functions into HTTP endpoints. Three routes:

- `GET /` — health check
- `POST /upload` — accept a file, save it, call `ingest_file()`
- `POST /ask` — accept `{"question": "..."}`, call `answer_question()`

Start the server:
```bash
cd app
bash run.sh
# → http://localhost:8000
# → http://localhost:8000/docs   ← interactive Swagger UI, free with FastAPI
```

### 4.5 `requirements.txt` — the libraries
Pinned versions so the code keeps working a year from now. The five that matter:

| Library | What it does |
|---|---|
| `langchain` | Glue. Defines `Document`, `Splitter`, prompt templates. |
| `faiss-cpu` | The vector store. |
| `sentence-transformers` | The local embedding model. |
| `fastapi` + `uvicorn` | The web server. |
| `pypdf`, `python-docx` | Read PDFs and Word docs. |

### 4.6 `.env.example` → `.env`
You **copy** this file to `.env` and fill in your real API key. `.env` is git-ignored. The pattern is universal in Python apps.

---

## 5. Run it yourself — 90 seconds

```bash
cd app
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env
# open .env, paste your Anthropic or OpenAI key
uvicorn main:app --reload
```

In another terminal:
```bash
# Upload a doc
curl -F "file=@handbook.pdf" http://localhost:8000/upload

# Ask
curl -X POST http://localhost:8000/ask \
     -H "Content-Type: application/json" \
     -d '{"question": "How many vacation days do new hires get?"}'
```

Or open `http://localhost:8000/docs` and click around — FastAPI gives you a UI for free.

---

## 6. The seven things students get wrong

1. **Skipping chunking.** Embedding a whole 50-page PDF as one vector throws away all the precision. Always chunk.
2. **Chunk size too big.** Above ~1500 chars and similarity search gets fuzzy. Stay around 500–1000.
3. **Forgetting overlap.** No overlap = answers get cut in half. Use 10–15% of chunk size.
4. **No `allow_dangerous_deserialization=True` when loading FAISS.** You'll get a confusing error. We set it because we trust our own files.
5. **Loading the embedding model on every request.** Slow. Cache it (we do — see `get_embeddings()`).
6. **Not pinning library versions.** LangChain breaks its own API roughly every six weeks. Use `requirements.txt` with `==`.
7. **Trusting the LLM to "know it doesn't know."** It won't, by default. The system prompt has to tell it, explicitly, what to say when the answer isn't there.

---

## 7. Where to go next

You now have the **minimum viable RAG**. Real production systems add:

- **Hybrid search** — combine vector similarity with old-school keyword (BM25) for better recall.
- **Reranking** — re-score the top 20 results with a smarter model, keep the top 4.
- **Chat history** — track previous turns so follow-up questions ("what about for managers?") work.
- **Streaming** — stream tokens to the UI as the LLM generates them.
- **Evals** — a fixed set of (question, expected-answer) pairs you re-run after every change.
- **Pinecone / Weaviate / Qdrant** — swap FAISS for a hosted store when your index outgrows one machine.
- **Guardrails** — input validation, PII redaction, refusal patterns.

Each of those is a single afternoon of work on top of what you've built. But none of them matter if the basics aren't solid — and now they are.

Welcome to applied LLMs. Go ship something.
