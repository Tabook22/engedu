# EngEdu Phase One Specification

## 1. Project Overview

EngEdu is an AI-powered English Speaking Tutor designed to help students improve spoken English through personalized assessment, adaptive learning, and future AI-driven conversation practice.

Phase One is limited to student understanding, speaking baseline assessment, personalized avatar profile creation, and professional assessment reporting. The avatar created in Phase One is **not yet a live conversational agent**. It is a stored profile and behavioral prompt foundation for later phases.

## 2. Phase One Objectives

Phase One must deliver a complete one-session assessment flow that:

1. Welcomes the student in a friendly and encouraging way.
2. Collects student discovery information through 7–10 interview questions.
3. Records the student reading three passages of increasing difficulty.
4. Analyzes interview responses and speaking performance.
5. Estimates the student's CEFR speaking baseline with a confidence score.
6. Generates a personalized `AvatarProfile` including a behavioral system prompt template.
7. Generates a professional structured assessment report.
8. Generates a downloadable PDF report.
9. Stores clear baseline measurements for future progress tracking.

## 3. Strict Scope

### In Scope

- Student identity capture.
- Assessment session lifecycle.
- Discovery interview questions and answers.
- Three reading passages: beginner, intermediate, upper-intermediate.
- Audio upload and secure storage.
- Transcription using Whisper or Faster-Whisper.
- Quantitative speaking metrics.
- Structured LLM qualitative analysis.
- CEFR scoring methodology.
- Avatar profile generation.
- Avatar behavioral prompt template generation.
- Assessment report JSON generation.
- PDF report generation and download.
- Security, deployment, database, API, and implementation documentation.

### Out of Scope

- Live AI conversation sessions.
- Real-time conversational avatar behavior.
- Real-time speech interruption or correction.
- Long-term analytics dashboard.
- Teacher/admin dashboard.
- Payments, subscriptions, or commercial plans.
- Multi-tenant school management.
- Production-scale background workers unless explicitly requested.

Later phases may be mentioned only at high level in `docs/roadmap.md`.

## 4. Required Technology Stack

### Backend

- Python 3.11+
- FastAPI
- Pydantic v2
- SQLAlchemy 2.x
- Alembic
- python-multipart
- SQLite for local development
- PostgreSQL-compatible schema for production

### Frontend

- React
- Vite
- TypeScript
- React Router
- TanStack Query recommended
- Axios or typed Fetch wrapper

### AI and Media

- OpenAI Whisper or Faster-Whisper for transcription.
- LLM with structured JSON output for qualitative analysis, avatar creation, and report drafting.
- ReportLab recommended for PDF generation; PyMuPDF acceptable.
- Pillow optional for future avatar visual placeholders.

## 5. User Roles

### Student

The student completes the assessment in one session. The experience must be friendly, patient, positive, and non-judgmental.

### Hermes Agent (engedu)

Hermes is the technical architect. Hermes prepares infrastructure plans, architecture, documentation, database design, API design, security planning, deployment planning, and implementation blueprints.

### Cursor

Cursor implements the system by reading the documentation in `docs/`. Cursor must not redesign the architecture unless Hermes explicitly updates the documentation.

## 6. Mandatory Cursor Rule

Before writing code, Cursor must read:

1. `docs/specification.md`
2. `docs/architecture.md`
3. `docs/roadmap.md`
4. `docs/database-schema.md`
5. `docs/api-endpoints.md`
6. `docs/security-checklist.md`
7. `docs/deployment-checklist.md`
8. `docs/prompts.md`

These documents are authoritative for Phase One.

## 7. Phase One Workflow

The application must follow this exact sequence.

### Step 1 — Welcome Screen

Display a welcoming introduction explaining:

- Purpose of the assessment.
- Estimated duration: maximum 10–12 minutes.
- What will be evaluated: interests, goals, confidence, reading aloud, pronunciation, fluency, vocabulary, grammar, and reading accuracy.
- How results will be used: to create a personalized speaking companion profile and baseline report.

The tone must always be positive, friendly, and encouraging.

Required student fields:

- Display name, required.
- Email, optional in Phase One.
- Native language, optional.
- Consent confirmation for audio recording and assessment processing, required.

### Step 2 — Student Discovery Interview

Ask 7–10 questions using multiple-choice, multi-select, scale, and open-ended formats.

Canonical questions:

1. Age group — single choice.
2. Education level — single choice.
3. Main English learning goal — single choice.
4. Current confidence speaking English — scale.
5. Preferred learning style — single or multi-select.
6. Favorite activities or hobbies — multi-select plus optional free text.
7. Tell us about yourself — open text.
8. Why are you learning English? — open text.
9. What topics do you enjoy discussing? — open text.
10. What would you like to achieve with English in the next 3–6 months? — open text.

Store all responses with raw and normalized values.

The objective is to understand:

- Personality.
- Interests.
- Hobbies.
- Motivation.
- Confidence.
- Communication preferences.
- Learning preferences.

### Step 3 — Speaking Assessment

Present three reading passages of increasing difficulty. The student records voice while reading each passage.

#### Passage 1: Beginner

> My name is Alex. I live in a small city with my family. I like learning English because I want to talk with people from other countries. Every day, I practice new words and simple sentences. I feel happy when I can speak clearly.

#### Passage 2: Intermediate

> Last weekend, I visited a new coffee shop near my school. The place was quiet, comfortable, and full of interesting books. I ordered tea, read a short story, and talked with a friend about our future plans. Experiences like this help me feel more confident when I speak English.

#### Passage 3: Upper-intermediate

> Communication is more than using correct words. When people speak a new language, they also learn how to express opinions, explain feelings, and respond naturally in different situations. Building confidence takes time, but regular practice, helpful feedback, and meaningful conversations can make progress easier to notice.

For each recording, store:

- Audio file relative path.
- Original filename if uploaded.
- MIME type.
- File size.
- Duration.
- Reading passage ID.
- Words per minute.
- Timestamp.
- Assessment session ID.

Recording requirements:

- Browser requests microphone permission.
- Student can re-record before submission.
- Accepted formats: `webm`, `wav`, `mp3`, `m4a` where browser/backend support exists.
- Maximum recording duration per passage: 90 seconds.
- Backend validates MIME type, extension, file size, and session relationship.

### Step 4 — AI Analysis

#### Interview Response Analysis

Extract:

- Interests.
- Hobbies.
- Personality traits.
- Communication style.
- Learning preferences.
- Motivation level.
- Confidence level.
- Goal categories.
- Suggested avatar type.

Use deterministic parsing plus LLM structured JSON output.

#### Speaking Performance Analysis

For each recording:

1. Transcribe audio using Whisper or Faster-Whisper.
2. Compare transcript to source passage.
3. Calculate quantitative metrics:
   - Duration.
   - Source word count.
   - Transcript word count.
   - Words per minute.
   - Pause frequency estimate.
   - Long pause count when segment timestamps are available.
   - Reading accuracy estimate.
   - Pronunciation accuracy estimate.
4. Run LLM structured qualitative analysis.
5. Score feedback categories:
   - Pronunciation.
   - Fluency.
   - Vocabulary.
   - Grammar.
   - Confidence.
   - Reading accuracy.

#### CEFR Scoring

Estimate speaking proficiency using CEFR levels: A1, A2, B1, B2, C1, C2.

Weighted final score:

```text
pronunciation_score  * 0.25
fluency_score        * 0.25
reading_accuracy     * 0.20
grammar_score        * 0.10
vocabulary_score     * 0.10
confidence_score     * 0.10
```

Suggested score mapping:

- 0–29: A1
- 30–44: A2
- 45–59: B1
- 60–74: B2
- 75–89: C1
- 90–100: C2

Confidence score should consider:

- All three recordings are present.
- Recording durations are reasonable.
- Transcription quality is acceptable.
- Quantitative and qualitative signals agree.
- Scores are consistent across passages.

Reports must state this is an estimated learning baseline, not an official CEFR certification.

### Step 5 — Personalized Avatar Creation

Generate and store an `AvatarProfile`.

Required data:

- Avatar name.
- Avatar type.
- Personality type.
- Conversation style.
- Interests.
- Encouragement style.
- Correction style.
- CEFR alignment.
- Language complexity rules.
- Feedback timing rules.
- Behavioral system prompt template.

Allowed starter avatar types:

- Explorer: curious and adventurous.
- Gamer: fun and energetic.
- Professional Coach: career-focused and structured.
- Friendly Mentor: supportive and confidence-building.
- Creative Partner: imaginative and topic-driven.

Critical behavioral rule: future avatar feedback must happen after the student finishes speaking. The avatar must not interrupt.

### Step 6 — Assessment Report

Generate a professional report containing:

- Student profile: name, date, interests, goals, motivation summary, learning preferences.
- English level: estimated CEFR level and confidence score.
- Strengths.
- Improvement areas.
- Recommendations.
- Suggested conversation topics.
- Avatar summary.
- Baseline measurements.

Generate both:

1. Structured report JSON.
2. Downloadable PDF report.

Baseline measurements must include:

- Overall CEFR estimate.
- CEFR confidence score.
- Final weighted score.
- Average WPM.
- WPM per passage.
- Reading accuracy per passage.
- Pronunciation score.
- Fluency score.
- Vocabulary score.
- Grammar score.
- Confidence score.
- Pause frequency estimate.
- Assessment date.

## 8. Functional Requirements

### FR-1 Student Creation

The system must create and retrieve student records with consent status.

### FR-2 Assessment Session Lifecycle

The system must create sessions and track status:

- `in_progress`
- `submitted`
- `processing`
- `completed`
- `failed`

The system must prevent processing until required interview answers and three recordings are present.

### FR-3 Interview Questions

The system must expose canonical Phase One interview questions from the backend so the frontend does not hard-code business logic.

### FR-4 Interview Answers

The system must save all interview answers, preserving raw values and normalized values.

### FR-5 Reading Passages

The system must expose the three required reading passages in order.

### FR-6 Audio Upload

The system must upload and store one recording per passage per session. Replacing a recording before processing is allowed.

### FR-7 Transcription

The system must transcribe each recording and store transcript text, model/provider metadata, and timestamps/segments when available.

### FR-8 Speaking Metrics

The system must calculate and persist per-passage and aggregate metrics.

### FR-9 Structured AI Analysis

LLM outputs must be validated against Pydantic schemas before persistence.

### FR-10 Avatar Profile

The system must generate exactly one active avatar profile for the completed assessment session.

### FR-11 Report Generation

The system must generate report JSON and PDF after successful analysis.

### FR-12 Report Download

The system must expose a secure PDF download endpoint.

## 9. Non-Functional Requirements

### UX

- Always friendly, encouraging, supportive, patient, and positive.
- Never use discouraging language.
- Never provide feedback before a student finishes speaking.
- Make errors understandable and non-threatening.

### Security

- Validate all inputs.
- Validate audio file MIME type, extension, and size.
- Sanitize filenames.
- Store generated filenames instead of trusting user filenames.
- Do not expose absolute filesystem paths.
- Keep secrets in environment variables.

### Privacy

- Require explicit consent for audio recording and AI assessment processing.
- Store audio securely.
- Define retention and deletion policies.
- Avoid logging sensitive audio/transcript/personal data in production.

### Reliability

- Failed processing must preserve submitted data.
- Processing should be retryable.
- Report generation should be deterministic from stored data.

### Maintainability

- Separate API routes, models, schemas, services, prompts, utilities, and tests.
- Keep prompts versionable and reusable.
- Keep frontend pages, components, hooks, API clients, and types separate.

## 10. User Stories

1. As a student, I want to understand what the assessment does before I begin so I feel comfortable.
2. As a student, I want to answer questions about my goals and interests so the system can personalize my experience.
3. As a student, I want to record myself reading short passages so my speaking level can be assessed.
4. As a student, I want the result to be encouraging so I feel motivated to continue learning.
5. As a student, I want to receive a clear report so I know my strengths and next steps.
6. As a future tutor system, I need a stored avatar profile so future speaking practice can be personalized.
7. As a developer, I need clear API, database, prompt, and architecture documentation so implementation is unambiguous.

## 11. Acceptance Criteria

Phase One is complete when:

1. Student can complete the full assessment in one session.
2. Welcome, interview, speaking, processing, and result steps are implemented in order.
3. All interview answers are saved.
4. All three audio recordings are saved and linked to the assessment session.
5. Transcriptions and speaking metrics are generated.
6. CEFR estimate and confidence score are generated.
7. Avatar profile and behavioral prompt template are stored.
8. Assessment report JSON is stored.
9. PDF report is downloadable.
10. Baseline measurements are persisted for future progress tracking.
11. Documentation is detailed enough for Cursor to implement without ambiguity.
