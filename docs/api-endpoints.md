# EngEdu Phase One API Endpoints

## 1. API Conventions

Base API path: `/api/v1`

General rules:

- Use JSON for standard requests/responses.
- Use `multipart/form-data` for audio upload.
- Use UUID strings for IDs.
- Do not expose absolute filesystem paths.
- Return consistent error objects.

## 2. Standard Error Shape

```json
{
  "detail": {
    "code": "ASSESSMENT_NOT_READY",
    "message": "Complete all required questions and recordings before submitting.",
    "fields": {
      "missing_question_keys": ["learning_goal"],
      "missing_passage_keys": ["upper_intermediate"]
    }
  }
}
```

Common error codes:

- `VALIDATION_ERROR`
- `STUDENT_NOT_FOUND`
- `SESSION_NOT_FOUND`
- `QUESTION_NOT_FOUND`
- `READING_PASSAGE_NOT_FOUND`
- `INVALID_SESSION_STATUS`
- `ASSESSMENT_NOT_READY`
- `INVALID_AUDIO_TYPE`
- `AUDIO_TOO_LARGE`
- `AUDIO_DURATION_TOO_LONG`
- `TRANSCRIPTION_FAILED`
- `AI_ANALYSIS_FAILED`
- `AVATAR_NOT_FOUND`
- `REPORT_NOT_FOUND`
- `PDF_GENERATION_FAILED`

## 3. Health

### GET /health

Checks backend health.

Response 200:

```json
{
  "status": "ok",
  "service": "engedu-backend"
}
```

Error example 500:

```json
{
  "detail": {
    "code": "SERVICE_UNHEALTHY",
    "message": "The service is not healthy."
  }
}
```

## 4. Create Student

### POST /api/v1/students

Creates a student identity record and records consent.

Request:

```json
{
  "display_name": "Mariam",
  "email": "mariam@example.com",
  "native_language": "Arabic",
  "consent_audio_recording": true,
  "consent_ai_analysis": true
}
```

Response 201:

```json
{
  "id": "2b9d8c50-1f32-4fd0-94de-82d08a1b1011",
  "display_name": "Mariam",
  "email": "mariam@example.com",
  "native_language": "Arabic",
  "consent_audio_recording": true,
  "consent_ai_analysis": true,
  "consent_timestamp": "2026-06-21T10:00:00Z",
  "created_at": "2026-06-21T10:00:00Z"
}
```

Error 422:

```json
{
  "detail": {
    "code": "VALIDATION_ERROR",
    "message": "Display name and consent are required."
  }
}
```

## 5. Get Student

### GET /api/v1/students/{student_id}

Response 200:

```json
{
  "id": "2b9d8c50-1f32-4fd0-94de-82d08a1b1011",
  "display_name": "Mariam",
  "email": "mariam@example.com",
  "native_language": "Arabic",
  "created_at": "2026-06-21T10:00:00Z"
}
```

Error 404:

```json
{
  "detail": {
    "code": "STUDENT_NOT_FOUND",
    "message": "Student was not found."
  }
}
```

## 6. Start Assessment

### POST /api/v1/assessment-sessions

Creates a new assessment session.

Request:

```json
{
  "student_id": "2b9d8c50-1f32-4fd0-94de-82d08a1b1011"
}
```

Response 201:

```json
{
  "id": "c36b27e2-a8b9-44d6-a04e-0e2018d12210",
  "student_id": "2b9d8c50-1f32-4fd0-94de-82d08a1b1011",
  "status": "in_progress",
  "started_at": "2026-06-21T10:01:00Z",
  "overall_cefr_level": null,
  "cefr_confidence": null
}
```

Error 409:

```json
{
  "detail": {
    "code": "CONSENT_REQUIRED",
    "message": "Audio recording and AI analysis consent are required before starting."
  }
}
```

## 7. Get Assessment Session

### GET /api/v1/assessment-sessions/{session_id}

Response 200:

```json
{
  "id": "c36b27e2-a8b9-44d6-a04e-0e2018d12210",
  "student_id": "2b9d8c50-1f32-4fd0-94de-82d08a1b1011",
  "status": "completed",
  "started_at": "2026-06-21T10:01:00Z",
  "submitted_at": "2026-06-21T10:10:00Z",
  "completed_at": "2026-06-21T10:12:00Z",
  "overall_cefr_level": "B1",
  "cefr_confidence": 0.78,
  "final_weighted_score": 57.4,
  "processing_error": null
}
```

## 8. Get Questions

### GET /api/v1/interview-questions

Returns active Phase One discovery questions.

Response 200:

```json
[
  {
    "id": "uuid",
    "question_key": "age_group",
    "question_text": "Which age group are you in?",
    "response_type": "single_choice",
    "options": ["Under 13", "13-17", "18-24", "25-34", "35+"],
    "display_order": 1,
    "is_required": true
  }
]
```

## 9. Save Answers

### PUT /api/v1/assessment-sessions/{session_id}/interview-answers

Creates or replaces all interview answers for a session.

Request:

```json
{
  "answers": [
    {
      "question_key": "age_group",
      "raw_answer": "18-24",
      "normalized_answer": "18–24"
    },
    {
      "question_key": "about_self",
      "raw_answer": "I am a university student and I like football.",
      "normalized_answer": "I am a university student and I like football."
    }
  ]
}
```

Response 200:

```json
{
  "assessment_session_id": "c36b27e2-a8b9-44d6-a04e-0e2018d12210",
  "count": 10,
  "answers": [
    {
      "id": "uuid",
      "question_key": "age_group",
      "normalized_answer": "18–24",
      "answered_at": "2026-06-21T10:03:00Z"
    }
  ]
}
```

Error 409:

```json
{
  "detail": {
    "code": "INVALID_SESSION_STATUS",
    "message": "Answers cannot be changed after processing has started."
  }
}
```

## 10. Get Saved Answers

### GET /api/v1/assessment-sessions/{session_id}/interview-answers

Response 200:

```json
[
  {
    "id": "uuid",
    "question_key": "age_group",
    "question_text": "Which age group are you in?",
    "response_type": "single_choice",
    "raw_answer": "18-24",
    "normalized_answer": "18–24",
    "display_order": 1
  }
]
```

## 11. Get Reading Passages

### GET /api/v1/reading-passages

Response 200:

```json
[
  {
    "id": "uuid",
    "passage_key": "beginner",
    "title": "Passage 1: Beginner",
    "difficulty": "Beginner",
    "display_order": 1,
    "text": "My name is Alex...",
    "word_count": 42
  }
]
```

## 12. Upload Audio

### POST /api/v1/assessment-sessions/{session_id}/audio-recordings

Uploads or replaces a recording for one passage.

Content-Type: `multipart/form-data`

Fields:

- `reading_passage_id`: UUID string, required.
- `duration_seconds`: float, required if available from browser.
- `recorded_at`: ISO datetime, optional.
- `file`: audio file, required.

Response 201:

```json
{
  "id": "uuid",
  "assessment_session_id": "c36b27e2-a8b9-44d6-a04e-0e2018d12210",
  "reading_passage_id": "uuid",
  "passage_key": "beginner",
  "mime_type": "audio/webm",
  "file_size_bytes": 842120,
  "duration_seconds": 31.2,
  "words_per_minute": 86.5,
  "uploaded_at": "2026-06-21T10:07:00Z"
}
```

Error 415:

```json
{
  "detail": {
    "code": "INVALID_AUDIO_TYPE",
    "message": "Unsupported audio type. Please upload webm, wav, mp3, or m4a."
  }
}
```

Error 413:

```json
{
  "detail": {
    "code": "AUDIO_TOO_LARGE",
    "message": "Audio file exceeds the maximum allowed size."
  }
}
```

## 13. List Audio Recordings

### GET /api/v1/assessment-sessions/{session_id}/audio-recordings

Response 200:

```json
[
  {
    "id": "uuid",
    "passage_key": "beginner",
    "difficulty": "Beginner",
    "duration_seconds": 31.2,
    "file_size_bytes": 842120,
    "words_per_minute": 86.5,
    "uploaded_at": "2026-06-21T10:07:00Z"
  }
]
```

## 14. Submit Assessment

### POST /api/v1/assessment-sessions/{session_id}/submit

Marks the assessment as ready for processing.

Request: empty body.

Response 200:

```json
{
  "id": "c36b27e2-a8b9-44d6-a04e-0e2018d12210",
  "status": "submitted",
  "missing_requirements": []
}
```

Error 409:

```json
{
  "detail": {
    "code": "ASSESSMENT_NOT_READY",
    "message": "Complete all required questions and recordings before submitting.",
    "fields": {
      "missing_question_keys": ["english_achievement_goal"],
      "missing_passage_keys": ["upper_intermediate"]
    }
  }
}
```

## 15. Analyze Speaking / Process Assessment

### POST /api/v1/assessment-sessions/{session_id}/analyze

Runs Phase One processing. This endpoint performs interview analysis, transcription, speaking analysis, CEFR scoring, avatar creation, and report generation.

Request: empty body.

Response 200 when synchronous processing completes:

```json
{
  "id": "c36b27e2-a8b9-44d6-a04e-0e2018d12210",
  "status": "completed",
  "overall_cefr_level": "B1",
  "cefr_confidence": 0.78,
  "avatar_profile_id": "uuid",
  "assessment_report_id": "uuid"
}
```

Response 202 if implemented asynchronously later:

```json
{
  "id": "c36b27e2-a8b9-44d6-a04e-0e2018d12210",
  "status": "processing"
}
```

Error 500:

```json
{
  "detail": {
    "code": "TRANSCRIPTION_FAILED",
    "message": "The audio could not be transcribed. Please try again or upload a clearer recording."
  }
}
```

## 16. Get Analysis Results

### GET /api/v1/assessment-sessions/{session_id}/analysis

Response 200:

```json
{
  "session_id": "c36b27e2-a8b9-44d6-a04e-0e2018d12210",
  "status": "completed",
  "profile_analysis": {
    "interests": ["football", "technology"],
    "goals": ["career communication"],
    "personality_traits": ["curious", "practical"],
    "communication_style": "friendly and direct",
    "learning_preferences": ["examples", "conversation practice"],
    "motivation_level": "high",
    "confidence_level": "medium",
    "summary": "Mariam is motivated by career growth and enjoys practical topics."
  },
  "speaking_analysis": {
    "overall_cefr_level": "B1",
    "cefr_confidence": 0.78,
    "final_weighted_score": 57.4,
    "category_scores": {
      "pronunciation": 62,
      "fluency": 55,
      "vocabulary": 58,
      "grammar": 54,
      "confidence": 60,
      "reading_accuracy": 70
    },
    "strengths": ["Good motivation", "Clear reading in the beginner passage"],
    "improvement_areas": ["Reduce long pauses", "Practice sentence rhythm"]
  }
}
```

## 17. Create Avatar

### POST /api/v1/assessment-sessions/{session_id}/avatar-profile

Creates avatar profile from existing completed analysis. The main analyze endpoint should call this internally, but this endpoint is useful for retrying avatar generation.

Request: empty body.

Response 201:

```json
{
  "id": "uuid",
  "avatar_name": "Coach Leo",
  "avatar_type": "Professional Coach",
  "personality_type": "Supportive, focused, and practical",
  "conversation_style": "Clear, structured, encouraging, and career-focused",
  "interests": ["technology", "career growth", "football"],
  "encouragement_style": "Celebrate small wins and remind the student that confidence grows with practice.",
  "correction_style": "Give one or two focused corrections after the student finishes speaking.",
  "cefr_alignment": "B1",
  "language_complexity_rules": "Use mostly B1 language with occasional B2 stretch phrases explained simply.",
  "feedback_timing_rules": "Do not interrupt while the student is speaking. Give feedback after completion.",
  "behavioral_prompt_template": "# Role\nYou are Coach Leo...",
  "is_active": true
}
```

Error 409:

```json
{
  "detail": {
    "code": "ANALYSIS_REQUIRED",
    "message": "Speaking and profile analysis must exist before creating an avatar."
  }
}
```

## 18. Get Avatar Profile

### GET /api/v1/assessment-sessions/{session_id}/avatar-profile

Response 200: same shape as create avatar response.

Error 404:

```json
{
  "detail": {
    "code": "AVATAR_NOT_FOUND",
    "message": "Avatar profile has not been generated yet."
  }
}
```

## 19. Generate Report

### POST /api/v1/assessment-sessions/{session_id}/report

Generates report JSON and PDF from completed analysis and avatar profile. The main analyze endpoint should call this internally, but this endpoint is useful for retrying report generation.

Request: empty body.

Response 201:

```json
{
  "id": "uuid",
  "assessment_session_id": "c36b27e2-a8b9-44d6-a04e-0e2018d12210",
  "report_title": "EngEdu Speaking Assessment Report",
  "generated_at": "2026-06-21T10:12:00Z",
  "download_url": "/api/v1/assessment-sessions/c36b27e2-a8b9-44d6-a04e-0e2018d12210/report/download"
}
```

Error 409:

```json
{
  "detail": {
    "code": "AVATAR_REQUIRED",
    "message": "Avatar profile must exist before generating the report."
  }
}
```

## 20. Get Report

### GET /api/v1/assessment-sessions/{session_id}/report

Response 200:

```json
{
  "id": "uuid",
  "report_title": "EngEdu Speaking Assessment Report",
  "generated_at": "2026-06-21T10:12:00Z",
  "student_profile": {
    "name": "Mariam",
    "date": "2026-06-21",
    "interests": ["football", "technology"],
    "goals": ["Speak confidently at work"]
  },
  "english_level": {
    "estimated_cefr_level": "B1",
    "confidence_score": 0.78,
    "limitations_note": "This is an estimated learning baseline, not an official CEFR certification."
  },
  "strengths": [],
  "improvement_areas": [],
  "recommendations": [],
  "baseline_measurements": {}
}
```

## 21. Download Report

### GET /api/v1/assessment-sessions/{session_id}/report/download

Downloads the generated PDF report.

Response 200:

Headers:

```text
Content-Type: application/pdf
Content-Disposition: attachment; filename="engedu-assessment-report.pdf"
```

Error 404:

```json
{
  "detail": {
    "code": "REPORT_NOT_FOUND",
    "message": "Assessment report has not been generated yet."
  }
}
```

## 22. Frontend Call Sequence

1. `POST /api/v1/students`
2. `POST /api/v1/assessment-sessions`
3. `GET /api/v1/interview-questions`
4. `PUT /api/v1/assessment-sessions/{session_id}/interview-answers`
5. `GET /api/v1/reading-passages`
6. `POST /api/v1/assessment-sessions/{session_id}/audio-recordings` three times
7. `POST /api/v1/assessment-sessions/{session_id}/submit`
8. `POST /api/v1/assessment-sessions/{session_id}/analyze`
9. `GET /api/v1/assessment-sessions/{session_id}/analysis`
10. `GET /api/v1/assessment-sessions/{session_id}/avatar-profile`
11. `GET /api/v1/assessment-sessions/{session_id}/report`
12. `GET /api/v1/assessment-sessions/{session_id}/report/download`
