## Project Architecture

### 🧩 Stack Summary
- **Languages:** TypeScript, Python  
- **Frontend:** React (Next.js), Tailwind CSS  
- **Backend:** NestJS / Fastify  
- **Async Tasks:** BullMQ (Redis)  
- **OCR:** Python (FastAPI + Tesseract / PaddleOCR)  
- **LLM / Embeddings:** Ollama (local, models `nomic-embed-text`, `bge-m3`, `llama3`)  
- **Databases:**
  - **Postgres** — metadata and documents  
  - **Qdrant** — vector storage  
  - **MinIO** — files  
  - **Redis** — task queue  
- **Containerization:** Docker Compose  
- **Monorepo:** Turborepo + pnpm workspaces  

---

### **apps/web (Next.js + React)**
- File upload (drag & drop), preview, processing status.
- Pages: document list, search, chat with citations, link to original (MinIO presigned URL).

### **apps/api (NestJS/Fastify)**
- `POST /upload` — saves file to MinIO, record to Postgres, task to Redis.
- `GET /search` — query embedding (Ollama), top-K from Qdrant, returns snippets.
- `POST /chat` — retriever → LLM (Ollama), response with citations (`docId/page/chunk`).
- `GET /presign/:fileKey` — provides temporary link to original.

### **apps/worker (Node + BullMQ)**
- Task processing:
  - load from MinIO → determine type (`pdf` / `image`);
  - PDF: text extraction (PyMuPDF / OCR);
  - OCR: request to `services/ocr`;
  - chunking (`packages/core`);
  - embeddings (Ollama);
  - upsert to Qdrant and metadata record to Postgres.

### **services/ocr (Python + FastAPI)**
- `POST /ocr` `{ fileKey | bytes, lang }` → `{ pages: [{ page, text, blocks }], confidence }`
- Uses Tesseract or PaddleOCR + preprocessing (OpenCV).

### **Infrastructure (Docker Compose)**
- **qdrant** — vector storage.  
- **postgres** — metadata database.  
- **minio** — object storage.  
- **redis** — task queue.  
- **ollama** — LLM and embeddings.  
- **ocr** — Python OCR service.
