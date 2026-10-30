# RAG API

A small retrieval-augmented generation (RAG) API built with FastAPI, ChromaDB, and Ollama. It indexes the paragraphs in `profile.txt`, retrieves the most relevant context for a question, and asks a local Ollama model to generate an answer.

## Requirements

- Python 3.10+
- [Ollama](https://ollama.com/) running locally
- The Ollama models `nomic-embed-text` and `qwen2.5:0.5b`

## Setup

1. Create and activate a virtual environment:

   ```powershell
   python -m venv venv
   .\venv\Scripts\Activate.ps1
   ```

2. Install the Python dependencies:

   ```powershell
   pip install -r requirements.txt
   ```

3. Pull the required Ollama models:

   ```powershell
   ollama pull nomic-embed-text
   ollama pull qwen2.5:0.5b
   ```

4. Build the local ChromaDB knowledge base:

   ```powershell
   python build_knowledge_base.py
   ```

5. Start the API:

   ```powershell
   uvicorn main:app --reload
   ```

The API is available at `http://127.0.0.1:8000`.

## Usage

Ask a question with the `question` query parameter:

```text
http://127.0.0.1:8000/ask?question=What are my career goals?
```

The response contains the question, the generated answer, and the context retrieved from the knowledge base.

## Project structure

| File | Purpose |
| --- | --- |
| `main.py` | FastAPI application and `/ask` endpoint |
| `build_knowledge_base.py` | Builds the ChromaDB collection from `profile.txt` |
| `profile.txt` | Source profile used as the knowledge base |
| `requirements.txt` | Direct Python dependencies |
| `chroma_db/` | Generated local vector database, ignored by Git |

## Notes

- Ollama must be running at its default address, `http://localhost:11434`.
- Run `build_knowledge_base.py` again after changing `profile.txt`.
- The generated `chroma_db/` directory is intentionally not committed; each developer can rebuild it locally.
