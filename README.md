# OCR RAG Document QA

A powerful self-hosted document question-answering system powered by OCR and RAG (Retrieval-Augmented Generation) technology. Upload documents, extract text using OCR, and ask questions with AI-powered answers backed by vector search.

## ✨ Features

- 📄 **Multi-format Support**: PDF and image document processing
- 🔍 **OCR Integration**: Tesseract and PaddleOCR for text extraction
- 🤖 **AI-Powered Q&A**: Local LLM integration with Ollama
- 🔎 **Vector Search**: Semantic search using Qdrant vector database
- 💬 **Chat Interface**: Interactive chat with document citations
- 📁 **File Management**: Drag & drop upload with MinIO storage
- ⚡ **Real-time Processing**: Asynchronous task processing with BullMQ
- 🐳 **Docker Ready**: Complete containerized setup

## 🧩 Tech Stack

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

## 🏗️ Architecture

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

## 🚀 Quick Start

### Prerequisites

- Docker and Docker Compose
- Node.js 18+ and pnpm
- Git

### Local Development

1. **Clone the repository**
   ```bash
   git clone https://github.com/amaksymov/ocr-rag-document-qa.git
   cd ocr-rag-document-qa
   ```

2. **Install dependencies**
   ```bash
   pnpm install
   ```

3. **Start infrastructure services**
   ```bash
   docker compose up -d
   ```

4. **Start development servers**
   ```bash
   pnpm turbo run dev --parallel
   ```

5. **Access the application**
   - Frontend: http://localhost:3001
   - API: http://localhost:3000
   - MinIO UI: http://localhost:9001
   - Qdrant UI: http://localhost:6333/dashboard
   - OCR API: http://localhost:8000

## 🐳 Production Deployment

### Using Docker Compose

1. **Build images**
   ```bash
   docker compose -f docker-compose.yml build
   ```

2. **Push to your registry**
   ```bash
   docker tag web yourregistry/web:latest
   docker push yourregistry/web:latest
   # (repeat for api, worker, ocr)
   ```

3. **Deploy on server**
   ```bash
   # Copy docker-compose.example.yml to your server
   # Configure .env with your variables (DB, keys, etc.)
   docker compose -f docker-compose.example.yml up -d
   ```

### Environment Variables

Create a `.env` file with the following variables:

```env
# Database
POSTGRES_URL=postgresql://user:password@localhost:5432/ocr_rag
REDIS_URL=redis://localhost:6379

# MinIO
MINIO_ENDPOINT=localhost
MINIO_PORT=9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin

# Qdrant
QDRANT_URL=http://localhost:6333

# Ollama
OLLAMA_BASE_URL=http://localhost:11434

# OCR Service
OCR_SERVICE_URL=http://localhost:8000
```

## 📖 Usage

1. **Upload Documents**: Drag and drop PDF or image files
2. **Wait for Processing**: Documents are automatically processed in the background
3. **Search**: Use the search interface to find relevant content
4. **Chat**: Ask questions about your documents and get AI-powered answers with citations

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- [Ollama](https://ollama.ai/) for local LLM hosting
- [Qdrant](https://qdrant.tech/) for vector search
- [Tesseract](https://github.com/tesseract-ocr/tesseract) and [PaddleOCR](https://github.com/PaddlePaddle/PaddleOCR) for OCR capabilities
