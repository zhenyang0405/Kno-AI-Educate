# Kno AI-Educate

**An AI-powered adaptive learning platform that transforms any PDF into a personalized study experience — with pre/post assessments, real-time AI tutoring via voice and screen sharing, and multi-agent orchestration across six microservices.**

[![Gemini](https://img.shields.io/badge/Model-Google%20Gemini%202.5-4285F4?logo=google&logoColor=white)](https://ai.google.dev/)
[![Google ADK](https://img.shields.io/badge/Framework-Google%20ADK-4285F4?logo=google&logoColor=white)](https://google.github.io/adk-docs/)
[![FastAPI](https://img.shields.io/badge/Backend-FastAPI-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![React](https://img.shields.io/badge/Frontend-React%2019-61DAFB?logo=react&logoColor=black)](https://react.dev/)
[![TypeScript](https://img.shields.io/badge/Language-TypeScript-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Python](https://img.shields.io/badge/Language-Python%203.11+-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![PostgreSQL](https://img.shields.io/badge/Database-PostgreSQL-4169E1?logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![Google Cloud Run](https://img.shields.io/badge/Infra-Cloud%20Run-4285F4?logo=googlecloud&logoColor=white)](https://cloud.google.com/run)
[![Firebase](https://img.shields.io/badge/Auth-Firebase-FFCA28?logo=firebase&logoColor=black)](https://firebase.google.com/)
[![Docker](https://img.shields.io/badge/Container-Docker-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![Tailwind CSS](https://img.shields.io/badge/Styling-Tailwind%20CSS%20v4-06B6D4?logo=tailwindcss&logoColor=white)](https://tailwindcss.com/)
[![License](https://img.shields.io/badge/License-MIT-green)](#license)

---

## Problem Statement

Students juggling work and education (especially part-time postgraduate students) face a common frustration: study sessions that feel inefficient. Generic learning tools don't adapt to individual pace, don't identify knowledge gaps before you start, and don't provide real-time feedback on what you actually struggle with.

Kno solves this by creating a **closed-loop learning pipeline**: upload a PDF, get assessed on what you already know, study with an AI tutor that sees your screen and talks to you in real-time, then validate your improvement with a targeted post-assessment — all personalized to your learning style, goals, and weak areas.

---

## Architecture Overview

Kno is built as **six independent microservices**, each handling a distinct stage of the learning pipeline. Services communicate via REST APIs, with a React frontend orchestrating the user journey.
<img width="2462" height="1728" alt="architecture" src="https://github.com/user-attachments/assets/511cdaa3-f2c3-4bf2-ab45-25266343f0b4" />

### Service Responsibilities

| Service | Port | Role |
|---------|------|------|
| **Backend API** | 8000 | Knowledge & document management, Firebase auth, Cloud Storage uploads |
| **Onboarding Agent** | 8001 | Conversational preference learning via Gemini ADK agent with tool-calling |
| **Pre-Assessment Agent** | 8002 | PDF-based MCQ generation (10 questions), automated marking with AI feedback |
| **Pre-Active-Learn Service** | 8003 | Material context caching (Gemini), concept extraction, study session lifecycle |
| **Live Tutoring Agent** | 8004 | Real-time bidirectional voice + screen sharing tutoring via WebSocket + Gemini 2.5 Native Audio |
| **Post-Assessment Agent** | 8005 | Targeted post-assessment generation focused on weak concepts, comparative feedback against pre-assessment |

### Data Flow

1. **Onboarding** → Agent discovers learning preferences via conversation → saves to `user_preferences` (JSONB)
2. **Upload** → PDF stored in GCS → metadata in `materials` table
3. **Pre-Assessment** → Agent reads PDF via GCS → generates 10 MCQs → user answers → AI marks and identifies weak areas
4. **Active Learning** → Service creates Gemini context cache from PDF → extracts concepts → Live agent uses cache + concepts + preferences for personalized tutoring
5. **Post-Assessment** → Agent generates harder questions targeting weak concepts → comparative scoring against pre-assessment

---

## Tech Stack

### Languages & Frameworks
- **Backend**: Python 3.11+, FastAPI, Google ADK (Agent Development Kit)
- **Frontend**: React 19, TypeScript, Tailwind CSS v4, Vite 7
- **Real-time**: WebSocket (native FastAPI), Web Audio API, MediaStream API

### AI & LLM
- **Google Gemini**: Flash, Pro, 2.5 Flash Native Audio (for real-time voice)
- **Google ADK**: Agent orchestration with tool-calling, session management
- **Gemini Context Caching**: Token-efficient repeated queries against study materials
- **Gemini Image Generation**: Visual aid creation during tutoring sessions

### Data & Storage
- **PostgreSQL** (Cloud SQL): Users, preferences, materials, assessments, questions, study sessions, concept tracking
- **Google Cloud Storage**: PDF material storage with signed URL access
- **Firebase Auth**: Anonymous and authenticated user sessions

### Infrastructure
- **Google Cloud Run**: All six microservices containerized and deployed independently
- **Firebase Hosting**: Static frontend deployment
- **Docker & Docker Compose**: Local development environment for each service

### Frontend Libraries
- **react-pdf** / **pdfjs-dist**: Split-pane PDF viewer
- **@excalidraw/excalidraw**: Collaborative whiteboard for visual thinking
- **react-markdown**: Rendering AI tutor responses with rich formatting

---

## Key Features

- **Multi-Agent Orchestration**: Six specialized AI agents (onboarding, question generation, assessment marking, concept extraction, live tutoring, image generation) collaborate across services using Google ADK with tool-calling — each agent has domain-specific system instructions and database tools.

- **Real-Time AI Tutoring with Voice & Screen Sharing**: Bidirectional WebSocket streams PCM audio at 16kHz (mic) and 24kHz (playback) between the browser and Gemini 2.5 Flash Native Audio. Screen frames are captured at 1 FPS via `getDisplayMedia`, JPEG-compressed, and sent as realtime image blobs — the AI tutor literally sees what you're reading and teaches accordingly.

- **Adaptive Assessment Pipeline**: Pre-assessment generates balanced-difficulty MCQs from your PDF. Post-assessment deliberately skews harder (4-5 hard questions vs 2-3 in pre) and targets your specific weak concepts identified during the study session. The AI marker compares both scores and generates structured JSON feedback with strengths, weaknesses, and recommendations.

- **Gemini Context Caching for Token Efficiency**: PDFs are uploaded to Gemini's File API and cached with `CreateCachedContentConfig`. All subsequent agent queries against the same material reuse the cache, significantly reducing token consumption and response latency for long documents.

- **Concept-Level Understanding Tracking**: The pre-active-learn service extracts 5-15 key concepts from each PDF (with page ranges and prerequisites) using Gemini. The live tutor updates `user_understanding` summaries per concept in real-time based on conversation evidence — not after every exchange, but at natural pause points when meaningful evidence accumulates.

---

## Getting Started

### Prerequisites

- **Python 3.11+**
- **Node.js 20+** and **npm**
- **Docker & Docker Compose** (for containerized local development)
- **PostgreSQL 15+** (local instance or Docker)
- **Google Cloud account** with:
  - Gemini API key (`GOOGLE_API_KEY`)
  - Cloud Storage bucket
  - Firebase project (for auth)

### Environment Setup

Each service has its own `.env` file. Here's the common structure:

```env
# Database (shared across all services)
DB_HOST=localhost
DB_NAME=db
DB_USER=user
DB_PASS=password
DB_PORT=5432

# Google Cloud
GOOGLE_API_KEY=your-gemini-api-key
GCS_BUCKET_NAME=your-bucket-name

# Firebase (backend service)
FIREBASE_SERVICE_ACCOUNT_PATH=serviceAccountKey.json
```

### Database Setup

```bash
# Start PostgreSQL (or use your existing instance)
# Then run the schema setup:
cd backend
python db_setup.py
```

This creates all required tables: `users`, `user_preferences`, `knowledge`, `materials`, `assessments`, `questions`, `user_answers`, plus indexes.

### Running Services Locally

**Backend API (port 8000):**
```bash
cd backend
pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

**Onboarding Agent (port 8001):**
```bash
cd onboarding-agent
pip install -r requirements.txt
uvicorn chat_agent.agent:app --host 0.0.0.0 --port 8001 --reload
```

**Pre-Assessment Agent (port 8002):**
```bash
cd study-session/pre-assessment
pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 8002 --reload
```

**Pre-Active-Learn Service (port 8003):**
```bash
cd study-session/pre-active-learn
pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 8003 --reload
```

**Live Tutoring Agent (port 8004):**
```bash
cd study-session/live-active-learning
pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 8004 --reload
```

**Post-Assessment Agent (port 8005):**
```bash
cd study-session/post-assessment
pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 8005 --reload
```

**Frontend:**
```bash
cd frontend
npm install
npm run dev
```

### Docker Compose (per service)

Each service includes a `docker-compose.yml` for containerized runs:

```bash
cd backend
docker-compose up --build
```

---

## API Documentation & Usage Examples

### 1. Create Knowledge Entry

```bash
curl -X POST http://localhost:8000/save-knowledge \
  -H "Authorization: Bearer <firebase-token>" \
  -F "name=Machine Learning Fundamentals" \
  -F "description=Stanford CS229 lecture notes"
```

**Response:**
```json
{
  "status": "success",
  "knowledge_id": "1"
}
```

### 2. Upload Study Material

```bash
curl -X POST http://localhost:8000/upload-document \
  -H "Authorization: Bearer <firebase-token>" \
  -F "knowledge_id=1" \
  -F "file=@lecture-notes.pdf"
```

### 3. Generate Pre-Assessment Questions

```bash
curl -X POST http://localhost:8002/api/pre-assessment/generate-questions \
  -H "Content-Type: application/json" \
  -d '{
    "material_id": 1,
    "storage_path": "users/uid123/knowledge/1/abc.pdf",
    "storage_bucket": "ai-educate-storage",
    "session_id": "session-1",
    "user_id": 1
  }'
```

**Response:**
```json
{
  "success": true,
  "material_id": 1,
  "questions_generated": 10,
  "message": "Successfully generated 10 questions",
  "assessment_type": "pre"
}
```

### 4. Onboarding Conversation

```bash
curl -X POST "http://localhost:8001/chat?uid=firebase-uid-123" \
  -H "Content-Type: application/json" \
  -d '{"message": "I prefer hands-on projects and visual learning"}'
```

**Response:**
```json
{
  "response": "That's great! Hands-on learning is one of the most effective approaches. I've noted your preference for visual and project-based learning. What topics are you most interested in exploring?"
}
```

### 5. Live Tutoring (WebSocket)

```javascript
// Connect to live tutoring
const ws = new WebSocket('ws://localhost:8004/ws/1/session-1');

// Send audio (PCM Int16, 16kHz, mono)
ws.send(audioBuffer); // ArrayBuffer

// Send screen capture
ws.send(JSON.stringify({
  type: 'image',
  mimeType: 'image/jpeg',
  data: base64ImageData
}));

// Receive audio response (PCM, 24kHz)
ws.onmessage = (event) => {
  if (event.data instanceof ArrayBuffer) {
    // Play PCM audio
  } else {
    // Parse JSON for transcriptions
    const data = JSON.parse(event.data);
  }
};
```

---

## Design Decisions & Trade-offs

### Microservices over Monolith
Each learning stage (onboarding, pre-assessment, active learning, post-assessment) has fundamentally different compute profiles. The live tutoring agent maintains long-lived WebSocket connections with audio streaming, while the question generator is a short-lived burst of LLM calls. Splitting into six services allows independent scaling, deployment, and failure isolation on Cloud Run. The trade-off is increased operational complexity and inter-service latency, which is acceptable given services don't need synchronous communication with each other.

### Google ADK for Agent Orchestration
Chose Google ADK over LangChain/LangGraph because: (1) native Gemini integration eliminates adapter overhead, (2) built-in tool-calling with automatic parameter extraction works reliably with Gemini models, (3) `InMemorySessionService` is sufficient for our per-request agent runs without needing external session stores. The trade-off is vendor lock-in to Google's ecosystem, but since the entire stack is already on GCP, this is an acceptable coupling.

### Raw PCM Audio over WebRTC
The live tutoring uses raw WebSocket binary frames for audio instead of WebRTC. This was a deliberate choice: (1) Gemini's native audio API expects PCM input/output, so WebRTC's codec negotiation adds unnecessary complexity, (2) `ScriptProcessorNode` at 16kHz gives us direct PCM access with minimal latency, (3) one-to-one tutoring doesn't need WebRTC's peer-to-peer or SFU capabilities. The trade-off is no built-in echo cancellation (handled by browser's `echoCancellation` constraint) and no adaptive bitrate.

### PostgreSQL JSONB for Flexible Schemas
User preferences, assessment summaries, and weak concepts are stored as JSONB rather than normalized tables. This allows the AI agents to evolve their output structure without schema migrations. For example, the assessment marker generates structured JSON summaries (`overall_performance`, `strengths`, `areas_for_improvement`, `recommendations`) that can be extended without altering the database. The trade-off is less queryable than normalized columns, mitigated by GIN indexes on JSONB fields.

### Anonymous Auth as Default
Firebase anonymous auth is used as the default authentication method. This eliminates signup friction for hackathon demos and allows immediate personalization via the onboarding agent. The trade-off is data persistence tied to device/browser session, acceptable for the current MVP stage.

---

## Performance & Scale Considerations

- **Context Cache Token Savings**: Gemini context caching for a 30-page PDF reduces token usage by approximately 80% on subsequent queries within the cache TTL, bringing per-interaction costs from ~$0.02 to ~$0.004.

- **Audio Streaming Latency**: The WebSocket audio pipeline achieves ~200-400ms round-trip for voice interactions. Audio chunks are 4096 samples at 16kHz (~256ms per chunk), and Gemini 2.5 Native Audio returns PCM at 24kHz that's scheduled for gapless playback using Web Audio API's `AudioBufferSourceNode` queue.

- **Screen Sharing Bandwidth**: Screen frames are captured at 1 FPS, downscaled to max 1024px dimension, and JPEG-compressed at 0.6 quality — resulting in ~50-100KB per frame. This keeps WebSocket bandwidth under 1Mbps even during active screen sharing.

- **Database Connection Pooling**: The backend API uses `psycopg2.pool.SimpleConnectionPool` (1-10 connections) to avoid connection overhead on Cloud Run cold starts. Each service manages its own pool sized to its expected concurrency.

- **Stateless Services**: All six services are stateless (session state lives in PostgreSQL, audio state lives in-memory per WebSocket connection). This enables Cloud Run's automatic horizontal scaling. The live tutoring agent is the exception — each WebSocket connection holds in-memory state, so scaling requires sticky sessions or connection draining.

---

## Future Improvements / Roadmap

- [ ] **AI Orchestrator Agent**: A meta-agent that guides students through concepts with hints, interactive animations, and adaptive pacing based on real-time understanding signals.
- [ ] **Collaborative Learning**: Multi-student study rooms where multiple users join the same AI-tutored session with shared whiteboard and group progress tracking.
- [ ] **Spaced Repetition System**: Leverage assessment results and concept mastery data to schedule review sessions at optimal intervals for long-term retention.
- [ ] **Multi-Modal Material Support**: Expand beyond PDFs to support lecture videos (with transcription), slides, and audio recordings as learning materials.
- [ ] **Progress Analytics Dashboard**: Concept mastery heatmaps, assessment score trends over time, and personalized study recommendations.
- [ ] **A2A Protocol Integration**: Revisit the Agent-to-Agent protocol for dynamic inter-agent discovery and collaboration without hardcoded orchestration.
- [ ] **Mobile-First Experience**: Responsive mobile interface with offline support for downloaded materials.
- [ ] **Redis Caching Layer**: Add Redis for session state, rate limiting, and frequently accessed data to reduce PostgreSQL load at scale.
- [ ] **gRPC Inter-Service Communication**: Migrate high-frequency internal API calls from REST to gRPC for lower latency and schema enforcement via Protocol Buffers.

---

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

## Acknowledgments

Built as a solo project for the **Gemini 3 Hackathon**, demonstrating that modern AI tooling empowers individual developers to build ambitious, production-deployed platforms.

**Tech Credits**: Google Gemini API, Google ADK, Google Cloud Run, Firebase, Excalidraw, react-pdf, Tailwind CSS.
