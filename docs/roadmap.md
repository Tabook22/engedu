# EngEdu Roadmap

## 1. Roadmap Rule

Build only Phase One now. Phases Two, Three, and Four are high-level placeholders only. Cursor must not implement later phases until Hermes explicitly creates detailed documentation for them.

## 2. Phase One — Student Profiling, Speaking Assessment, Avatar Foundation

### Goal

Create a complete one-session assessment flow that deeply understands the student, estimates their English speaking baseline, creates a personalized avatar profile, and generates a professional assessment report.

### Student Outcome

The student completes a 10–12 minute friendly assessment and receives:

- Estimated CEFR speaking level.
- Confidence score.
- Strengths.
- Improvement areas.
- Practical recommendations.
- Personalized avatar profile preview.
- Downloadable PDF report.

### Milestone 1 — Documentation Foundation

Tasks:

1. Create `docs/specification.md`.
2. Create `docs/architecture.md`.
3. Create `docs/roadmap.md`.
4. Create `docs/database-schema.md`.
5. Create `docs/api-endpoints.md`.
6. Create `docs/security-checklist.md`.
7. Create `docs/deployment-checklist.md`.
8. Create `docs/prompts.md`.

Completion criteria:

- Cursor has enough documentation to implement Phase One without redesigning.

### Milestone 2 — Project Foundation

Tasks:

1. Create repository structure.
2. Create backend FastAPI app.
3. Create frontend Vite app.
4. Add `.env.example`.
5. Add `README.md`.
6. Add `AGENTS.md` instructing Cursor to follow docs.
7. Add backend and frontend test foundations.

Completion criteria:

- Backend starts locally.
- Frontend starts locally.
- `GET /health` succeeds.

### Milestone 3 — Database Foundation

Tasks:

1. Configure SQLAlchemy.
2. Configure Alembic.
3. Implement models from `docs/database-schema.md`.
4. Create initial migration.
5. Seed interview questions.
6. Seed reading passages.

Completion criteria:

- Empty SQLite database migrates successfully.
- Required tables, indexes, constraints, questions, and passages exist.

### Milestone 4 — Student and Session APIs

Tasks:

1. Implement create student endpoint.
2. Implement get student endpoint.
3. Implement start assessment endpoint.
4. Implement get assessment status endpoint.
5. Implement submit assessment endpoint.
6. Add session lifecycle validation.

Completion criteria:

- Student can start an assessment session.
- Session statuses transition correctly.

### Milestone 5 — Interview Flow APIs

Tasks:

1. Implement get interview questions endpoint.
2. Implement save answers endpoint.
3. Implement get answers endpoint.
4. Validate 7–10 answer requirement.
5. Store raw and normalized answer values.

Completion criteria:

- Student interview answers are saved and retrieved correctly.

### Milestone 6 — Reading Passage and Audio APIs

Tasks:

1. Implement get reading passages endpoint.
2. Implement audio upload endpoint.
3. Validate MIME type, extension, size, duration, and session.
4. Store generated audio file paths.
5. Implement list recordings endpoint.

Completion criteria:

- Student can upload exactly one current recording per passage.
- Invalid uploads are rejected safely.

### Milestone 7 — Transcription and Metrics

Tasks:

1. Implement transcription provider abstraction.
2. Implement Faster-Whisper or Whisper integration.
3. Store transcripts and segments.
4. Implement WPM calculation.
5. Implement reading accuracy calculation.
6. Implement pause estimate calculation.
7. Add unit tests for metrics.

Completion criteria:

- All three recordings can produce transcript and metrics records.

### Milestone 8 — AI Analysis and CEFR Scoring

Tasks:

1. Implement interview profile analysis prompt.
2. Implement speaking analysis prompt.
3. Validate LLM structured JSON outputs.
4. Implement CEFR scoring service.
5. Store profile and speaking analysis.
6. Store final CEFR estimate and confidence score on session.

Completion criteria:

- Completed assessment has category scores, CEFR estimate, confidence score, strengths, and improvement areas.

### Milestone 9 — Avatar Profile Generation

Tasks:

1. Implement avatar generation prompt.
2. Generate avatar name, type, personality, style, interests, encouragement style, correction style, CEFR alignment, and behavioral prompt.
3. Validate prompt template contains required sections.
4. Store one active avatar profile.

Completion criteria:

- Each completed assessment produces one stored `AvatarProfile` with behavioral template.

### Milestone 10 — Assessment Report Generation

Tasks:

1. Implement report generation prompt.
2. Create report JSON.
3. Create baseline metrics JSON.
4. Generate PDF report.
5. Implement report retrieval endpoint.
6. Implement PDF download endpoint.

Completion criteria:

- Student can view structured results and download PDF report.

### Milestone 11 — Frontend Assessment Experience

Tasks:

1. Build Welcome page.
2. Build Discovery Interview page.
3. Build Speaking Assessment page.
4. Build Processing page.
5. Build Results page.
6. Integrate API client.
7. Add progress stepper.
8. Add friendly error states.

Completion criteria:

- Student can complete the full workflow through UI.

### Milestone 12 — End-to-End Verification

Manual test:

1. Start backend.
2. Start frontend.
3. Create student and consent.
4. Start assessment.
5. Answer discovery questions.
6. Record three passages.
7. Submit assessment.
8. Process assessment.
9. Confirm CEFR estimate appears.
10. Confirm avatar profile appears.
11. Download PDF report.
12. Confirm database relationships are correct.
13. Confirm API responses do not expose absolute paths.

Completion criteria:

- Phase One acceptance criteria pass.

## 3. Phase Two — Speaking Practice Sessions, High Level Only

Goal: use the stored `AvatarProfile` to guide structured speaking practice sessions.

Possible future scope:

- Start practice session.
- Select topic based on avatar and interests.
- Record student responses.
- Give post-response feedback.
- Store session summaries.

Do not implement in Phase One.

## 4. Phase Three — AI Conversation Tutor, High Level Only

Goal: turn the avatar into an active AI conversation tutor.

Possible future scope:

- Conversational turn-taking.
- Speech-to-text and text-to-speech.
- Avatar response generation.
- Adaptive difficulty.
- Topic continuity.

Do not implement in Phase One.

## 5. Phase Four — Advanced Learning Analytics, High Level Only

Goal: track measurable progress over time.

Possible future scope:

- WPM trends.
- Pronunciation trend estimates.
- Fluency trend estimates.
- CEFR progression.
- Practice consistency analytics.
- Student dashboard.
- Teacher/admin dashboard later.

Do not implement in Phase One.

## 6. Phase One Definition of Done

- Required documentation exists.
- Backend tests pass.
- Frontend build passes.
- Full assessment can be completed in one session.
- All required data is stored with correct relationships.
- Avatar profile and prompt template are generated.
- Assessment report JSON and PDF are generated.
- Baseline measurements are stored for future tracking.
- No detailed later-phase implementation is built.
