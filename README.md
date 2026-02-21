# Document Intelligence Pipeline

A FastAPI-based service for uploading documents, extracting content, performing AI-powered analysis, and streaming real-time progress updates to clients via Server-Sent Events (SSE).

---

## Table of Contents

- [Features](#features)  
- [Architecture Overview](#architecture-overview)  
- [Setup & Run Instructions](#setup--run-instructions)  
- [Design Decisions](#design-decisions)  
- [Time Breakdown](#time-breakdown)  
- [Usage Example](#usage-example)  

---

## Features

- Upload PDF and TXT documents  
- Validate file type and size (up to 10MB)  
- Background processing pipeline with a job queue  
- Text extraction from PDFs and TXT files  
- AI-powered analysis using OpenAI GPT-4.1  
  - Generates concise summary (3–5 sentences)  
  - Extracts key topics/themes  
  - Determines sentiment (positive, negative, neutral, mixed)  
  - Identifies actionable items/recommendations  
- Real-time status updates via SSE  
- Persistent storage of document metadata and analysis results  

---

## Architecture Overview




**Components:**

1. **Queue:**  
   - Implemented with `asyncio.Queue` for lightweight in-memory job management.  
   - Each document upload creates a job with `file_id` and `user_id`.

2. **Worker:**  
   - Continuously consumes jobs from the queue.  
   - Extracts text content (PDF/TXT) and sends it to OpenAI GPT-4.1.  
   - Updates document status at each stage: `pending`, `processing`, `analyzing`, `completed`/`failed`.

3. **SSE Streaming:**  
   - Clients connect once to receive real-time status updates for all their documents.  
   - Each event contains `document_id`, `status`, `timestamp`, and `result` if available.  
   - Connection remains open until the client disconnects.  

4. **Persistence:**  
   - Document metadata and analysis results are stored in a JSON file (`documents.json`).  
   - Simple `Database` class ensures atomic reads and writes for reliability.  

---

## Setup & Run Instructions

1. **Clone the repository:**

```bash
git clone <repository_url>
cd document-analyzer
```

2. **Create and activate a virtual environment:**
```bash
python -m venv .venv
source .venv/bin/activate  # Linux/macOS
# or
.venv\Scripts\activate     # Windows
```

3. **Install dependencies:**
```bash
pip install -r requirements.txt
```

4. **Run the FastAPI server:**
```bash
uvicorn src.app.main:app --reload
```

5. **Endpoints:**
- Login: `POST /auth/login`
- Upload document: `POST /documents/upload`
- Get Document by Id: `GET /documents/{document_id}`
- Get All Documents: `GET /documents/`
- Delete Document: `DELETE /documents/{document_id}`
- Get Document status: `GET /documents/{document_id}/status`
- SSE status stream: `GET /documents/stream`
