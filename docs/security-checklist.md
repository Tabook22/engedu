# EngEdu Security Checklist

## 1. Security Scope

This checklist covers Phase One: student profiling, audio upload, speech assessment, avatar profile generation, and PDF report generation.

Student data, interview answers, transcripts, audio recordings, and reports are private learning records and must be protected.

## 2. Authentication Recommendations

Phase One may be implemented without full accounts for local MVP, but production should add authentication before public release.

Recommended progression:

- Local MVP: anonymous session IDs for testing only.
- Private beta: magic-link login or email-based authentication.
- Production: secure user accounts or SSO if used by schools.

Checklist:

- [ ] Do not expose student records by guessable IDs.
- [ ] Use UUIDs for public identifiers.
- [ ] Add authentication before public internet launch.
- [ ] Use secure cookies or bearer tokens when auth is added.
- [ ] Enable HTTPS in production.

## 3. Authorization Recommendations

Checklist:

- [ ] A student can access only their own assessment sessions.
- [ ] A report download must verify ownership/session authorization.
- [ ] Admin/developer retry endpoints must not be public in production.
- [ ] Future teacher/admin roles must use role-based access control.
- [ ] Avoid returning private data in list endpoints without authorization.

## 4. Consent and Student Protection

Checklist:

- [ ] Require explicit consent before microphone recording.
- [ ] Require explicit consent before AI analysis.
- [ ] Explain how audio and answers will be used.
- [ ] If used with minors, add guardian consent workflow before production.
- [ ] Provide clear privacy copy in the UI.
- [ ] Document retention/deletion policy.

Minimum consent fields:

- `consent_audio_recording`
- `consent_ai_analysis`
- `consent_timestamp`

## 5. Input Validation

Checklist:

- [ ] Validate all request bodies with Pydantic.
- [ ] Validate UUID path parameters.
- [ ] Validate email format when provided.
- [ ] Validate answer count: 7–10 answers.
- [ ] Validate answer question keys.
- [ ] Validate answer types match question definitions.
- [ ] Limit open-text answer length.
- [ ] Reject unknown enum values.
- [ ] Normalize but preserve raw answers.

Recommended limits:

- Display name: 1–120 characters.
- Email: 255 characters.
- Open answer: 2,000 characters.
- Multi-select answers: 20 selected values max.

## 6. File Upload Security — Audio

Audio upload is the highest-risk Phase One input area.

Checklist:

- [ ] Accept only expected audio extensions: `.webm`, `.wav`, `.mp3`, `.m4a`.
- [ ] Validate MIME type against configured allowlist.
- [ ] Enforce maximum upload size, recommended 25 MB.
- [ ] Enforce maximum duration, recommended 90 seconds per passage.
- [ ] Generate backend filenames; never trust user filenames for storage.
- [ ] Sanitize original filenames if stored for display.
- [ ] Store outside frontend static directory.
- [ ] Do not execute uploaded files.
- [ ] Do not pass untrusted file paths to shell commands.
- [ ] Store only relative file paths in database.
- [ ] Prevent path traversal using safe path joins.
- [ ] Consider scanning uploads in production if exposed publicly.

## 7. PDF Security

Checklist:

- [ ] Generate PDFs server-side from trusted structured report data.
- [ ] Sanitize text inserted into reports.
- [ ] Store PDFs outside frontend source directory.
- [ ] Download endpoint must authorize access.
- [ ] Use generated filenames for downloads.
- [ ] Do not embed secrets or internal file paths in PDFs.

## 8. LLM and Prompt Security

Checklist:

- [ ] Treat student input as untrusted content inside prompts.
- [ ] Use structured JSON output validation.
- [ ] Do not let student answers override system instructions.
- [ ] Validate CEFR values, scores, and required keys.
- [ ] Retry invalid JSON once with a repair prompt if needed.
- [ ] Store prompt versions or enough metadata for auditability.
- [ ] Do not log full sensitive prompt payloads in production.
- [ ] Do not claim official CEFR certification.

## 9. Secrets Management

Checklist:

- [ ] Store API keys in environment variables.
- [ ] Never commit `.env` files.
- [ ] Provide `.env.example` without real secrets.
- [ ] Use separate secrets for local, staging, and production.
- [ ] Rotate keys if accidentally exposed.
- [ ] Restrict API key permissions where provider supports it.

Required secret-like variables:

```text
OPENAI_API_KEY=
DATABASE_URL=
SECRET_KEY= # when authentication is introduced
```

## 10. Rate Limiting

Recommended before public release:

- [ ] Limit student/session creation per IP.
- [ ] Limit audio uploads per session.
- [ ] Limit analyze endpoint calls per session.
- [ ] Limit report download frequency.
- [ ] Add global request rate limits at Nginx or application layer.

Suggested starting limits:

- 10 assessment starts per IP per hour.
- 6 audio uploads per session per hour.
- 3 analysis retries per session.
- 20 report downloads per session per hour.

## 11. Logging Recommendations

Checklist:

- [ ] Log request IDs and session IDs.
- [ ] Log processing state transitions.
- [ ] Log transcription/analysis errors.
- [ ] Do not log raw audio contents.
- [ ] Avoid logging full transcripts in production.
- [ ] Avoid logging full LLM prompts in production.
- [ ] Redact emails and personal data where practical.
- [ ] Use structured logs in production.

## 12. Database Security

Checklist:

- [ ] Use migrations for schema changes.
- [ ] Use parameterized SQL through ORM.
- [ ] Do not construct raw SQL from user input.
- [ ] Use PostgreSQL user with least required privileges in production.
- [ ] Back up database regularly.
- [ ] Encrypt disk/volume in production if possible.

## 13. CORS and Browser Security

Checklist:

- [ ] Restrict CORS origins to configured frontend domains.
- [ ] Do not use wildcard CORS in production.
- [ ] Enable HTTPS in production.
- [ ] Add security headers through Nginx.
- [ ] Use secure cookies when authentication is added.

Recommended headers:

```text
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: strict-origin-when-cross-origin
Content-Security-Policy: default-src 'self'
```

## 14. Privacy and Retention

Recommended policy for Phase One:

- Audio files: retain until student deletes account/session or for a defined period such as 30–90 days.
- Transcripts and reports: retain as learning records until deletion request.
- Logs: retain minimal operational logs; avoid sensitive content.

Checklist:

- [ ] Document retention policy in README/deployment notes.
- [ ] Add deletion workflow in a future phase.
- [ ] Keep storage organized by student/session for future deletion.
- [ ] Avoid copying private audio into temporary public directories.

## 15. Production Hardening Checklist

Before public launch:

- [ ] Add authentication.
- [ ] Add authorization.
- [ ] Enable HTTPS.
- [ ] Configure CORS strictly.
- [ ] Use PostgreSQL.
- [ ] Use persistent encrypted storage.
- [ ] Add automated backups.
- [ ] Add Nginx upload size limits.
- [ ] Add application upload size limits.
- [ ] Add rate limiting.
- [ ] Add monitoring and alerts.
- [ ] Add structured logging.
- [ ] Add error tracking.
- [ ] Add privacy policy and consent text.
- [ ] Add data deletion procedure.
- [ ] Review LLM provider data handling terms.
