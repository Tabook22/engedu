# EngEdu Phase One Database Schema

## 1. Database Principles

- Use SQLAlchemy 2.x ORM and Alembic migrations.
- Use SQLite first for development.
- Keep schema PostgreSQL-compatible.
- Use UUID string primary keys for portability.
- Store audio and PDF files on disk; store relative paths in database.
- Store raw submitted data and normalized analysis data separately.
- Use UTC timestamps.
- Add `created_at` and `updated_at` to all core tables.

## 2. Entity Relationships

```text
Students
  └── AssessmentSessions
        ├── InterviewAnswers
        │     └── InterviewQuestions
        ├── AudioRecordings
        │     └── ReadingPassages
        ├── SpeakingAnalysis
        ├── AvatarProfiles
        └── AssessmentReports
```

## 3. Tables

## 3.1 Students

Table name: `students`

Purpose: stores student identity and consent metadata.

| Column | Type | Required | Notes |
|---|---:|---:|---|
| id | string UUID | yes | Primary key |
| display_name | varchar(120) | yes | Student-facing name |
| email | varchar(255) | no | Optional in Phase One |
| native_language | varchar(120) | no | Optional |
| consent_audio_recording | boolean | yes | Must be true before assessment |
| consent_ai_analysis | boolean | yes | Must be true before assessment |
| consent_timestamp | datetime | yes | UTC |
| created_at | datetime | yes | UTC |
| updated_at | datetime | yes | UTC |

Indexes:

- `ix_students_email` on `email`.

Constraints:

- `display_name` length > 0.
- Consent fields must be true before starting an assessment.

## 3.2 AssessmentSessions

Table name: `assessment_sessions`

Purpose: tracks one Phase One assessment attempt.

| Column | Type | Required | Notes |
|---|---:|---:|---|
| id | string UUID | yes | Primary key |
| student_id | string UUID | yes | FK to `students.id` |
| status | varchar(30) | yes | `in_progress`, `submitted`, `processing`, `completed`, `failed` |
| started_at | datetime | yes | UTC |
| submitted_at | datetime | no | UTC |
| processing_started_at | datetime | no | UTC |
| completed_at | datetime | no | UTC |
| processing_error | text | no | Developer-facing failure reason |
| overall_cefr_level | varchar(2) | no | A1–C2 |
| cefr_confidence | float | no | 0.0–1.0 |
| final_weighted_score | float | no | 0–100 |
| created_at | datetime | yes | UTC |
| updated_at | datetime | yes | UTC |

Indexes:

- `ix_assessment_sessions_student_id`
- `ix_assessment_sessions_status`
- `ix_assessment_sessions_started_at`

Constraints:

- `status` must be valid enum value.
- `cefr_confidence` between 0 and 1 when present.
- `final_weighted_score` between 0 and 100 when present.

## 3.3 InterviewQuestions

Table name: `interview_questions`

Purpose: stores canonical Phase One discovery questions.

| Column | Type | Required | Notes |
|---|---:|---:|---|
| id | string UUID | yes | Primary key |
| question_key | varchar(80) | yes | Stable key |
| question_text | text | yes | Display text |
| response_type | varchar(40) | yes | `single_choice`, `multi_choice`, `scale`, `open_text` |
| options_json | JSON/text | no | For choice questions |
| display_order | integer | yes | 1–10 |
| is_required | boolean | yes | Default true |
| is_active | boolean | yes | Default true |
| created_at | datetime | yes | UTC |
| updated_at | datetime | yes | UTC |

Indexes:

- Unique `question_key`.
- `ix_interview_questions_display_order`.

Canonical keys:

1. `age_group`
2. `education_level`
3. `english_learning_goal`
4. `speaking_confidence`
5. `preferred_learning_style`
6. `favorite_activities_hobbies`
7. `about_self`
8. `learning_motivation`
9. `discussion_topics`
10. `english_achievement_goal`

## 3.4 InterviewAnswers

Table name: `interview_answers`

Purpose: stores student answers for a session.

| Column | Type | Required | Notes |
|---|---:|---:|---|
| id | string UUID | yes | Primary key |
| assessment_session_id | string UUID | yes | FK to `assessment_sessions.id` |
| interview_question_id | string UUID | yes | FK to `interview_questions.id` |
| question_key | varchar(80) | yes | Snapshot key for auditability |
| question_text | text | yes | Snapshot text shown to student |
| response_type | varchar(40) | yes | Snapshot response type |
| raw_answer_json | JSON/text | yes | Exact submitted value |
| normalized_answer | text | no | Human-readable normalized answer |
| display_order | integer | yes | Snapshot order |
| answered_at | datetime | yes | UTC |
| created_at | datetime | yes | UTC |
| updated_at | datetime | yes | UTC |

Indexes:

- `ix_interview_answers_session_id`
- Unique `(assessment_session_id, interview_question_id)`.
- Unique `(assessment_session_id, question_key)`.

## 3.5 ReadingPassages

Table name: `reading_passages`

Purpose: stores required speaking assessment passages.

| Column | Type | Required | Notes |
|---|---:|---:|---|
| id | string UUID | yes | Primary key |
| passage_key | varchar(80) | yes | `beginner`, `intermediate`, `upper_intermediate` |
| title | varchar(160) | yes | Display title |
| difficulty | varchar(80) | yes | Beginner, Intermediate, Upper-intermediate |
| display_order | integer | yes | 1–3 |
| text | text | yes | Source passage |
| word_count | integer | yes | Precomputed |
| target_cefr_floor | varchar(2) | no | e.g. A1 |
| target_cefr_ceiling | varchar(2) | no | e.g. B2 |
| is_active | boolean | yes | Default true |
| created_at | datetime | yes | UTC |
| updated_at | datetime | yes | UTC |

Indexes:

- Unique `passage_key`.
- Unique `display_order` for active Phase One records.

Seed data:

- `beginner`, order 1, Beginner.
- `intermediate`, order 2, Intermediate.
- `upper_intermediate`, order 3, Upper-intermediate.

Use exact passage text from `docs/specification.md`.

## 3.6 AudioRecordings

Table name: `audio_recordings`

Purpose: stores metadata for uploaded audio files.

| Column | Type | Required | Notes |
|---|---:|---:|---|
| id | string UUID | yes | Primary key |
| assessment_session_id | string UUID | yes | FK to `assessment_sessions.id` |
| reading_passage_id | string UUID | yes | FK to `reading_passages.id` |
| storage_path | varchar(500) | yes | Relative path only |
| original_filename | varchar(255) | no | Sanitized display only |
| mime_type | varchar(120) | yes | Validated |
| file_extension | varchar(20) | yes | Validated |
| file_size_bytes | integer | yes | > 0 |
| duration_seconds | float | no | > 0 and <= 90 |
| words_per_minute | float | no | Source words / duration * 60 |
| recorded_at | datetime | no | UTC |
| uploaded_at | datetime | yes | UTC |
| created_at | datetime | yes | UTC |
| updated_at | datetime | yes | UTC |

Indexes:

- `ix_audio_recordings_session_id`
- `ix_audio_recordings_reading_passage_id`
- Unique `(assessment_session_id, reading_passage_id)`.

Constraints:

- `file_size_bytes > 0`.
- `duration_seconds > 0` when present.
- `duration_seconds <= 90` when present.

## 3.7 Transcriptions

Recommended additional table: `transcriptions`

Purpose: stores speech-to-text result for each audio recording. Although not named in the prompt's minimum list, it is necessary for clean implementation and auditability.

| Column | Type | Required | Notes |
|---|---:|---:|---|
| id | string UUID | yes | Primary key |
| audio_recording_id | string UUID | yes | FK to `audio_recordings.id` |
| provider | varchar(80) | yes | `faster_whisper`, `openai_whisper` |
| model_name | varchar(120) | yes | Model identifier |
| language | varchar(20) | no | Detected/configured language |
| transcript_text | text | yes | Full transcript |
| segments_json | JSON/text | no | Segment timestamps if available |
| confidence | float | no | 0–1 if available |
| processing_duration_seconds | float | no | Runtime |
| created_at | datetime | yes | UTC |
| updated_at | datetime | yes | UTC |

Indexes:

- Unique `audio_recording_id`.

## 3.8 SpeakingAnalysis

Table name: `speaking_analysis`

Purpose: stores per-session speaking analysis, category scores, CEFR estimate, and feedback.

| Column | Type | Required | Notes |
|---|---:|---:|---|
| id | string UUID | yes | Primary key |
| assessment_session_id | string UUID | yes | FK to `assessment_sessions.id` |
| transcription_summary_json | JSON/text | yes | Per-passage transcript references/summaries |
| passage_metrics_json | JSON/text | yes | Per-passage metrics |
| pronunciation_score | float | yes | 0–100 |
| fluency_score | float | yes | 0–100 |
| vocabulary_score | float | yes | 0–100 |
| grammar_score | float | yes | 0–100 |
| confidence_score | float | yes | 0–100 |
| reading_accuracy_score | float | yes | 0–100 |
| average_wpm | float | no | Average across passages |
| pause_frequency_estimate | float | no | Aggregate estimate |
| overall_cefr_level | varchar(2) | yes | A1–C2 |
| cefr_confidence | float | yes | 0–1 |
| final_weighted_score | float | yes | 0–100 |
| strengths_json | JSON/text | yes | List of strengths |
| improvement_areas_json | JSON/text | yes | List of improvement areas |
| recommendations_json | JSON/text | yes | List of recommendations |
| feedback_summary | text | yes | Encouraging summary |
| raw_llm_output_json | JSON/text | no | Full structured LLM output |
| created_at | datetime | yes | UTC |
| updated_at | datetime | yes | UTC |

Indexes:

- Unique `assessment_session_id`.

Constraints:

- All score columns between 0 and 100.
- `cefr_confidence` between 0 and 1.

## 3.9 ProfileAnalysis

Recommended additional table: `profile_analysis`

Purpose: stores structured analysis of discovery interview answers. This keeps student profiling separate from speaking analysis.

| Column | Type | Required | Notes |
|---|---:|---:|---|
| id | string UUID | yes | Primary key |
| assessment_session_id | string UUID | yes | FK to `assessment_sessions.id` |
| interests_json | JSON/text | yes | List |
| hobbies_json | JSON/text | yes | List |
| goals_json | JSON/text | yes | List |
| personality_traits_json | JSON/text | yes | List with evidence |
| communication_style | varchar(160) | no | Inferred style |
| learning_preferences_json | JSON/text | yes | List |
| motivation_level | varchar(40) | no | low/medium/high |
| confidence_level | varchar(40) | no | low/medium/high |
| suggested_avatar_type | varchar(80) | no | Explorer/Gamer/etc. |
| summary | text | yes | Human-readable summary |
| raw_llm_output_json | JSON/text | no | Full structured LLM output |
| created_at | datetime | yes | UTC |
| updated_at | datetime | yes | UTC |

Indexes:

- Unique `assessment_session_id`.

## 3.10 AvatarProfiles

Table name: `avatar_profiles`

Purpose: stores personalized avatar profile and future behavioral prompt template.

| Column | Type | Required | Notes |
|---|---:|---:|---|
| id | string UUID | yes | Primary key |
| student_id | string UUID | yes | FK to `students.id` |
| assessment_session_id | string UUID | yes | FK to `assessment_sessions.id` |
| avatar_name | varchar(120) | yes | Friendly name |
| avatar_type | varchar(80) | yes | Explorer, Gamer, Professional Coach, Friendly Mentor, Creative Partner |
| personality_type | varchar(200) | yes | Description |
| conversation_style | varchar(240) | yes | Future style |
| interests_json | JSON/text | yes | Topics to use later |
| encouragement_style | text | yes | How avatar encourages |
| correction_style | text | yes | How avatar corrects |
| cefr_alignment | varchar(2) | yes | A1–C2 |
| language_complexity_rules | text | yes | CEFR-appropriate guidance |
| feedback_timing_rules | text | yes | Must say feedback after student finishes speaking |
| behavioral_prompt_template | text | yes | Full future system prompt |
| visual_descriptor_json | JSON/text | no | Optional future visual hints |
| is_active | boolean | yes | Default true |
| created_at | datetime | yes | UTC |
| updated_at | datetime | yes | UTC |

Indexes:

- `ix_avatar_profiles_student_id`
- Unique `assessment_session_id`.

Service rule:

- Only one active avatar profile per student for Phase One unless future phases add avatar history management.

## 3.11 AssessmentReports

Table name: `assessment_reports`

Purpose: stores structured report and PDF metadata.

| Column | Type | Required | Notes |
|---|---:|---:|---|
| id | string UUID | yes | Primary key |
| student_id | string UUID | yes | FK to `students.id` |
| assessment_session_id | string UUID | yes | FK to `assessment_sessions.id` |
| avatar_profile_id | string UUID | no | FK to `avatar_profiles.id` |
| report_title | varchar(200) | yes | e.g. EngEdu Speaking Assessment Report |
| report_json | JSON/text | yes | Full structured report |
| baseline_measurements_json | JSON/text | yes | Future progress baseline |
| pdf_storage_path | varchar(500) | no | Relative path only |
| generated_at | datetime | yes | UTC |
| created_at | datetime | yes | UTC |
| updated_at | datetime | yes | UTC |

Indexes:

- `ix_assessment_reports_student_id`
- Unique `assessment_session_id`.

## 4. Required Enums

### AssessmentSessionStatus

```text
in_progress
submitted
processing
completed
failed
```

### ResponseType

```text
single_choice
multi_choice
scale
open_text
```

### CEFRLevel

```text
A1
A2
B1
B2
C1
C2
```

## 5. Baseline Measurements JSON

`assessment_reports.baseline_measurements_json` must include:

```json
{
  "assessment_date": "2026-06-21T00:00:00Z",
  "overall_cefr_level": "B1",
  "cefr_confidence": 0.78,
  "final_weighted_score": 57.4,
  "average_wpm": 92.5,
  "category_scores": {
    "pronunciation": 62,
    "fluency": 55,
    "vocabulary": 58,
    "grammar": 54,
    "confidence": 60,
    "reading_accuracy": 70
  },
  "passage_metrics": [
    {
      "passage_key": "beginner",
      "duration_seconds": 31.2,
      "words_per_minute": 86.5,
      "reading_accuracy_score": 76,
      "pronunciation_estimate_score": 68,
      "pause_frequency_estimate": 4.2
    }
  ]
}
```

## 6. Migration Requirements

Initial migration must:

1. Create all Phase One tables.
2. Create constraints and indexes.
3. Seed canonical interview questions.
4. Seed the three reading passages.
5. Be reversible where practical.

## 7. Data Integrity Rules

- Cannot create assessment without student consent.
- Cannot submit assessment unless required interview answers and three audio recordings exist.
- Cannot process assessment unless status is `submitted` or `failed` retry.
- Cannot create avatar profile unless profile and speaking analyses exist.
- Cannot create report unless avatar profile and speaking analysis exist.
- API must not return absolute storage paths.
