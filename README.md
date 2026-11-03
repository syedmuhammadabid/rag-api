# Local RAG API with FastAPI, ChromaDB, and Ollama

A lightweight **retrieval-augmented generation (RAG) API** that
combines FastAPI, ChromaDB, and Ollama to answer questions from private,
locally stored profile data. The project demonstrates local embeddings,
semantic search, metadata filtering, and local large language model (LLM)
responses without sending application data to a hosted AI API.

## Features

- Local RAG pipeline: retrieve relevant text, augment the prompt, and generate
  an answer.
- ChromaDB persistent vector storage for semantic search.
- Ollama embeddings with `nomic-embed-text`.
- Local answer generation with `qwen2.5:0.5b`.
- FastAPI endpoints for adding profile documents and asking questions.
- Optional user-level filtering when querying multiple profiles.
- Persistent local storage under `./chroma_db`.
- Automatic interactive API documentation through FastAPI.

## How it works

1. `profile.txt` is split into paragraph-sized chunks.
2. ChromaDB creates and stores embeddings for each chunk.
3. `/ask` retrieves the two most relevant chunks for a question.
4. Ollama uses the retrieved context to generate a grounded answer.

The API also supports adding additional profiles through `/documents`; those
documents are stored with a `user_name` metadata field and can be queried with
the optional `user` filter.

The application creates or reuses the ChromaDB collection named
`personal_profile`. Both the embedding model and the answer-generation model
run through the local Ollama service; no hosted LLM provider is configured.

## Requirements

- Python 3.10 or newer
- [Ollama](https://ollama.com/) installed and running locally
- Ollama models:
  - [`nomic-embed-text`](https://ollama.com/library/nomic-embed-text)
  - [`qwen2.5:0.5b`](https://ollama.com/library/qwen2.5)

## Quick start

### 1. Clone the repository

```powershell
git clone https://github.com/syedmuhammadabid/rag-api.git
cd rag-api
```

### 2. Create a virtual environment

```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
```

On macOS or Linux, activate the environment with:

```bash
source venv/bin/activate
```

### 3. Install dependencies

```powershell
pip install -r requirements.txt
```

### 4. Start Ollama and download the models

Make sure Ollama is running at its default address,
`http://localhost:11434`, then run:

```powershell
ollama pull nomic-embed-text
ollama pull qwen2.5:0.5b
```

### 5. Build the knowledge base

```powershell
python build_knowledge_base.py
```

This creates the local `chroma_db/` vector database from `profile.txt`.

### 6. Start the API

```powershell
uvicorn main:app --reload
```

The API is available at <http://127.0.0.1:8000>.

## API usage

### Ask a question

```bash
curl "http://127.0.0.1:8000/ask?question=What%20are%20my%20career%20goals%3F"
```

Optionally limit retrieval to one profile:

```bash
curl "http://127.0.0.1:8000/ask?question=What%20are%20your%20skills%3F&user=Abid"
```

Example response:

```json
{
  "question": "What are my career goals?",
  "answer": "The retrieved profile says ...",
  "context_used": ["..."],
  "filtered_by_user": null
}
```

### Add a profile document

```bash
curl -X POST "http://127.0.0.1:8000/documents" \
  -H "Content-Type: application/json" \
  -d '{"user_name":"Ada","content":"Ada builds reliable systems."}'
```

The `content` field is split into chunks at blank lines. Each chunk is stored
with `source: "profile"`, the submitted `user_name`, and a zero-based
`chunk_index` as ChromaDB metadata. The endpoint returns the number of chunks
added:

```json
{
  "message": "Added 1 chunks for user 'Ada'.",
  "user_name": "Ada",
  "chunks_added": 1
}
```

### Endpoint reference

| Method | Path | Parameters or body | Behavior |
| --- | --- | --- | --- |
| `GET` | `/ask` | Required `question`; optional `user` query parameter | Retrieves up to two matching chunks and generates an answer |
| `POST` | `/documents` | JSON body with required `user_name` and `content` strings | Splits content by blank lines and stores the chunks |

This example application has no authentication, authorization, rate limiting,
or request-size limits. Run it locally or add those controls before exposing it
to an untrusted network.

### Interactive API documentation

When the server is running, open:

- Swagger UI: <http://127.0.0.1:8000/docs>
- ReDoc: <http://127.0.0.1:8000/redoc>

## Project structure

| File or directory | Purpose |
| --- | --- |
| [`main.py`](./main.py) | FastAPI application with `/documents` and `/ask` |
| [`build_knowledge_base.py`](./build_knowledge_base.py) | Builds the ChromaDB collection from `profile.txt` |
| [`profile.txt`](./profile.txt) | Example source content for the knowledge base |
| [`requirements.txt`](./requirements.txt) | Pinned Python dependencies |
| `chroma_db/` | Generated local vector database; not committed to Git |

## Updating the knowledge base

Edit `profile.txt`, remove the generated database, and rebuild it:

```powershell
Remove-Item -Recurse -Force .\chroma_db
python build_knowledge_base.py
```

Run the rebuild before restarting the API so the collection contains the
latest source content.

The build script uses deterministic IDs such as `chunk0`, while `/documents`
uses IDs based on the submitted user name. Reusing the same user name and chunk
indexes can cause duplicate-ID errors in ChromaDB; use unique user names or
rebuild the database when replacing the sample data.

## Troubleshooting

- **Ollama connection errors:** confirm Ollama is running and available at
  `http://localhost:11434`.
- **Model errors:** run both `ollama pull` commands from the setup steps.
- **Stale or duplicate ChromaDB data:** remove `chroma_db/` and run the
  knowledge-base build script again.
- **API import errors:** activate the virtual environment and reinstall
  dependencies with `pip install -r requirements.txt`.

## Contributing

Contributions are welcome. To propose an improvement:

1. Fork the repository.
2. Create a focused feature branch.
3. Make and test your change locally.
4. Open a pull request with a clear description of the change.

Please avoid committing personal data, model credentials, virtual
environments, or generated vector database files.

## License

No license has been included in this repository yet. Add a license file before
redistributing the project or accepting external contributions under defined
terms.
