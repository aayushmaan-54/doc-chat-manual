# Doc Chat (Manual RAG)
A command-line app that answers questions about your PDFs and cites the page each answer came from. Built by hand with no RAG frameworks or vector databases, to learn how retrieval-augmented generation works under the hood.

```
$ python ingest.py docs/
Ingested 2 files · 148 pages · 412 chunks

You: what is the refund window for annual plans?
AI:  Annual plans can be refunded within 30 days of purchase [pricing-policy.pdf, p. 4].

You: what about monthly?
AI:  Monthly plans aren't refundable, but you can cancel before the next billing date [pricing-policy.pdf, p. 5].

You: who is the CEO?
AI:  I couldn't find that in the documents.
```

## Goal
Learn every step of a RAG pipeline by implementing it yourself: loading, chunking, embeddings, similarity search, grounded answers with citations, and measuring retrieval quality.

## Rules
- Use the official SDK (`openai` / Azure OpenAI) directly for chat and embeddings.
- Store everything in SQLite.
- Keep it a CLI.

## Tech stack
Python, `openai` SDK, `pypdf`, `numpy`, SQLite (`sqlite3`), `pydantic-settings` for config.

## Things to implement
### 1. Ingestion
- [ ] `python ingest.py <file or folder>` loads PDFs and extracts text page by page
- [ ] Clean up whitespace and skip empty pages
- [ ] Hash each file; re-ingesting an unchanged file does nothing, a changed file replaces its old chunks

### 2. Chunking
- [ ] Write your own chunker with `CHUNK_SIZE` (e.g. 1000 chars) and `CHUNK_OVERLAP` (e.g. 200)
- [ ] Prefer breaking at paragraph or sentence boundaries instead of mid-word
- [ ] Keep the file name and page number on every chunk, so citations work later

### 3. Embeddings
- [ ] Embed chunks with the embeddings endpoint, in batches
- [ ] Store vectors in SQLite as BLOBs (numpy `float32` bytes), along with the embedding model name
- [ ] Retry rate limits and errors with backoff

### 4. Retrieval
- [ ] Embed the question and compute cosine similarity against all chunk vectors in numpy
- [ ] Return the top `TOP_K` (e.g. 4) chunks
- [ ] Drop chunks below `MIN_SCORE` so weak matches don't reach the prompt
- [ ] In debug mode, print the retrieved chunks with their scores

### 5. Answering
- [ ] Put retrieved chunks in the prompt as numbered, labeled sources
- [ ] Answer only from the provided sources, citing them as `[file, p. N]`
- [ ] Say "I couldn't find that in the documents" when the sources don't contain the answer
- [ ] Stream the answer to the terminal
- [ ] Keep the answer prompt in `prompts/`, not in code

### 6. Follow-up questions
- [ ] Keep chat history for the session
- [ ] Before searching, rewrite follow-ups into standalone questions ("what about monthly?" → "what is the refund policy for monthly plans?")
- [ ] Log the rewritten query so you can see what was actually searched

### 7. Retrieval evals
- [ ] Write `evals/questions.json`: about 15 questions, each with the file and page that contain the answer
- [ ] `python evals/run.py` reports hit rate: how often the right page appears in the top-k results
- [ ] Include a few questions worded very differently from the PDF text

### 8. Logging
- [ ] Log every call to `doc-chat-manual.log`: latency, tokens in/out, retrieved chunk IDs and scores
