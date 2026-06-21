# EngEdu Phase One Architecture

## 1. System Overview

EngEdu Phase One is a full-stack web application for student profiling, English speaking baseline assessment, personalized avatar profile generation, and assessment report generation.

The frontend guides the student through a supportive assessment experience. The backend stores data, processes audio, runs transcription and AI analysis, creates the avatar profile, and generates reports.

## 2. Component Diagram

```text
Student Browser
  |
  | React + Vite + TypeScript
  | REST JSON + multipart audio upload
  v
FastAPI Backend
  |
  |-- API Routes
  |-- Pydantic Schemas
  |-- SQLAlchemy Models
  |-- Assessment Services
  |-- Audio Storage Service
  |-- Transcription Service
  |-- Speaking Metrics Service
  |-- LLM Analysis Service
  |-- Avatar Generation Service
  |-- PDF Report Service
  |
  +--> SQLite Development DB
  +--> PostgreSQL Production DB
  +--> Local File Storage: audio + reports
  +--> Whisper/Faster-Whisper
  +--> LLM Structured JSON API
```

## 3. Required Project Structure

```text
project/
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── config.py
│   │   ├── database.py
│   │   ├── dependencies.py
│   │   ├── exceptions.py
│   │   ├── api/
│   │   │   ├── router.py
│   │   │   └── routes/
│   │   │       ├── health.py
│   │   │       ├── students.py
│   │   │       ├── assessment_sessions.py
│   │   │       ├── interview_questions.py
│   │   │       ├── interview_answers.py
│   │   │       ├── reading_passages.py
│   │   │       ├── audio_recordings.py
│   │   │       ├── analysis.py
│   │   │       ├── avatar_profiles.py
│   │   │       └── reports.py
│   │   ├── models/
│   │   ├── schemas/
│   │   ├── services/
│   │   ├── prompts/
│   │   └── utils/
│   ├── alembic/
│   ├── storage/
│   │   ├── audio/
│   │   └── reports/
│   ├── tests/
│   ├── pyproject.toml
│   └── alembic.ini
├── frontend/
│   ├── src/
│   │   ├── api/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── pages/
│   │   ├── types/
│   │   └── styles/
│   ├── package.json
│   └── vite.config.ts
├── docs/
├── scripts/
├── deployment/
├── tests/
├── prompts/
├── .env.example
├── README.md
└── AGENTS.md
```

## 4. Backend Architecture

### API Layer

API routes must be thin. They validate input, call services, and return response schemas.

Route groups:

- `health.py`: health checks.
- `students.py`: create/get student.
- `assessment_sessions.py`: start, submit, process, retry, status.
- `interview_questions.py`: canonical questions.
- `interview_answers.py`: save/get answers.
- `reading_passages.py`: required passages.
- `audio_recordings.py`: upload/list audio.
- `analysis.py`: retrieve speaking/profile analysis.
- `avatar_profiles.py`: retrieve generated avatar profile.
- `reports.py`: retrieve report JSON and download PDF.

### Service Layer

Services own business logic.

- `assessment_service.py`: session lifecycle and processing orchestration.
- `student_service.py`: student creation and validation.
- `interview_service.py`: question definitions and answer persistence.
- `storage_service.py`: safe file paths, saving files, deletion helpers.
- `transcription_service.py`: Whisper/Faster-Whisper integration.
- `speech_metrics_service.py`: WPM, reading accuracy, pause estimates.
- `profile_analysis_service.py`: interview response analysis.
- `speaking_analysis_service.py`: qualitative speaking analysis.
- `cefr_scoring_service.py`: final level and confidence scoring.
- `avatar_generation_service.py`: avatar profile and prompt generation.
- `report_generation_service.py`: report JSON creation.
- `pdf_report_service.py`: PDF rendering.

### Model Layer

SQLAlchemy models represent tables documented in `docs/database-schema.md`.

### Schema Layer

Pydantic schemas define:

- Request bodies.
- Response bodies.
- AI structured output contracts.
- Report JSON contracts.

## 5. Frontend Architecture

### Pages

- `WelcomePage`: explains assessment and collects student identity/consent.
- `DiscoveryInterviewPage`: displays 7–10 questions and saves answers.
- `SpeakingAssessmentPage`: records three reading passages.
- `ProcessingPage`: triggers/polls analysis.
- `ResultsPage`: shows CEFR estimate, avatar summary, report link.

### Components

- `Layout`
- `ProgressStepper`
- `WelcomeCard`
- `QuestionCard`
- `AudioRecorder`
- `PassageReader`
- `LoadingAnalysis`
- `ResultSummary`
- `AvatarPreview`
- `ReportDownloadButton`

### API Client Modules

- `client.ts`
- `students.ts`
- `assessments.ts`
- `questions.ts`
- `audio.ts`
- `analysis.ts`
- `avatars.ts`
- `reports.ts`

## 6. Data Flow

```text
Welcome
  -> create Student
  -> create AssessmentSession
  -> fetch InterviewQuestions
  -> save InterviewAnswers
  -> fetch ReadingPassages
  -> upload AudioRecording x3
  -> submit AssessmentSession
  -> process AssessmentSession
      -> analyze interview answers
      -> transcribe audio
      -> calculate speech metrics
      -> run speaking analysis
      -> calculate CEFR
      -> generate AvatarProfile
      -> generate AssessmentReport JSON
      -> generate PDF
  -> show Results
  -> download PDF
```

## 7. Audio Pipeline Architecture

### 7.1 Recording

- Frontend uses browser `MediaRecorder` when available.
- Student can listen and re-record before upload.
- Frontend sends `duration_seconds`, `recorded_at`, `reading_passage_id`, and audio file.

### 7.2 Upload Validation

Backend validates:

- Assessment session exists.
- Session status allows upload.
- Passage exists.
- MIME type is allowed.
- File extension is allowed.
- File size is under configured limit.
- Duration is positive and not above 90 seconds.

### 7.3 Storage

Storage path pattern:

```text
backend/storage/audio/<student_id>/<assessment_session_id>/<passage_key>.<extension>
```

Database stores only relative paths:

```text
audio/<student_id>/<assessment_session_id>/<passage_key>.webm
```

### 7.4 Transcription

- Preferred local provider: Faster-Whisper.
- Optional cloud provider: OpenAI Whisper.
- Store transcript text, provider, model, detected language, segments JSON, confidence if available, and processing duration.

### 7.5 Metrics

Calculate:

- Duration.
- Source word count.
- Transcript word count.
- WPM.
- Reading accuracy via text similarity.
- Pause frequency using segment timestamps when available.
- Pronunciation estimate from transcript mismatch plus LLM assessment.

### 7.6 Analysis

LLM receives structured input:

- Source passages.
- Transcripts.
- Metrics.
- Profile summary.
- CEFR rubric.

LLM returns validated structured JSON.

### 7.7 Error Handling

If a step fails:

- Mark session `failed` if processing cannot continue.
- Store `processing_error` for developers.
- Do not delete uploaded audio or submitted answers.
- Return student-friendly error messages.
- Allow retry from stored data.

## 8. Assessment Processing Orchestration

```text
submit session
  -> validate completeness
  -> set status submitted
process session
  -> set status processing
  -> profile analysis
  -> for each recording:
       transcribe
       calculate metrics
  -> aggregate metrics
  -> speaking analysis
  -> CEFR scoring
  -> avatar generation
  -> report JSON generation
  -> PDF generation
  -> set status completed
```

For Phase One MVP, processing may be synchronous. The architecture should permit background jobs later.

## 9. Local Development Workflow

```text
Developer machine
  -> clone repository
  -> create backend virtual environment
  -> install backend dependencies
  -> run Alembic migrations
  -> start FastAPI on localhost:8000
  -> install frontend dependencies
  -> start Vite on localhost:5173
  -> test end-to-end assessment flow
```

Recommended local URLs:

- Frontend: `http://localhost:5173`
- Backend: `http://localhost:8000`
- API docs: `http://localhost:8000/docs`

## 10. GitHub Workflow

- `main`: stable branch.
- `develop`: integration branch if needed.
- Feature branches: `feature/phase-one-<area>`.
- Pull requests must reference relevant docs.
- Cursor implementations must not diverge from docs without Hermes update.

Recommended checks:

- Backend tests.
- Backend lint/type checks.
- Frontend build.
- Frontend lint.

## 11. VPS Workflow

High-level deployment path:

```text
GitHub repository
  -> VPS pulls release branch/tag
  -> install backend dependencies
  -> run migrations
  -> build frontend
  -> serve frontend via Nginx
  -> reverse proxy /api to FastAPI
  -> persist DB and storage volumes
```

Production should use PostgreSQL and persistent storage. Detailed checklist is in `docs/deployment-checklist.md`.

## 12. Hermes Responsibilities

Hermes owns:

- Documentation.
- Architecture.
- Database design.
- API design.
- Security plan.
- Deployment plan.
- GitHub workflow plan.
- VPS preparation plan.
- Cursor handoff instructions.

## 13. Cursor Responsibilities

Cursor owns:

- Backend implementation.
- Frontend implementation.
- Database migrations.
- API implementation.
- UI components.
- Business logic.
- Tests.

Cursor must follow docs as source of truth.

## 14. Privacy Architecture

- Consent must be captured before recording.
- Audio files and reports are private student data.
- Store relative file paths only.
- Avoid logging raw transcripts/audio payloads in production.
- Add future retention/deletion support.
- Use least-privilege production file permissions.

## 15. Environment Variables

Required backend environment variables:

```text
APP_ENV=development
DATABASE_URL=sqlite:///./engedu.db
STORAGE_ROOT=./storage
MAX_AUDIO_UPLOAD_MB=25
ALLOWED_AUDIO_MIME_TYPES=audio/webm,audio/wav,audio/mpeg,audio/mp4,audio/x-m4a
TRANSCRIPTION_PROVIDER=faster_whisper
WHISPER_MODEL_SIZE=base
LLM_PROVIDER=openai
LLM_MODEL=gpt-4o-mini
OPENAI_API_KEY=
CORS_ALLOWED_ORIGINS=http://localhost:5173
REPORT_RETENTION_DAYS=365
AUDIO_RETENTION_DAYS=365
```

## 16. Architecture Decisions

1. Store audio and PDFs on filesystem for Phase One simplicity.
2. Store only relative paths in DB for portability.
3. Use structured JSON LLM outputs for reliability.
4. Keep canonical questions/passages backend-driven.
5. Treat CEFR result as an estimated baseline only.
6. Keep avatar as stored profile and prompt template, not a runtime agent.
