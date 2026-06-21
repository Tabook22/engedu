# EngEdu Deployment Checklist

## 1. Deployment Scope

This checklist prepares EngEdu Phase One for local development, GitHub workflow, and later VPS deployment. It does not require implementing full production automation during Phase One unless explicitly requested.

## 2. Local Development Setup

### Backend

Checklist:

- [ ] Create `backend/` directory.
- [ ] Create Python virtual environment.
- [ ] Install dependencies from `pyproject.toml`.
- [ ] Copy `.env.example` to `.env`.
- [ ] Configure SQLite `DATABASE_URL`.
- [ ] Configure `STORAGE_ROOT`.
- [ ] Configure transcription provider.
- [ ] Configure LLM provider/API key if cloud LLM is used.
- [ ] Run Alembic migrations.
- [ ] Start FastAPI server.

Recommended commands:

```bash
cd backend
python3 -m venv .venv
source .venv/bin/activate
pip install -e .
alembic upgrade head
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### Frontend

Checklist:

- [ ] Create `frontend/` Vite React TypeScript app.
- [ ] Install dependencies.
- [ ] Configure API base URL.
- [ ] Start Vite dev server.

Recommended commands:

```bash
cd frontend
npm install
npm run dev
```

Local URLs:

- Frontend: `http://localhost:5173`
- Backend: `http://localhost:8000`
- Backend docs: `http://localhost:8000/docs`

## 3. Required Environment Variables

Root `.env.example` or backend `.env.example` should include:

```text
APP_ENV=development
DATABASE_URL=sqlite:///./engedu.db
STORAGE_ROOT=./storage
MAX_AUDIO_UPLOAD_MB=25
ALLOWED_AUDIO_MIME_TYPES=audio/webm,audio/wav,audio/mpeg,audio/mp4,audio/x-m4a
ALLOWED_AUDIO_EXTENSIONS=.webm,.wav,.mp3,.m4a
MAX_RECORDING_DURATION_SECONDS=90
TRANSCRIPTION_PROVIDER=faster_whisper
WHISPER_MODEL_SIZE=base
LLM_PROVIDER=openai
LLM_MODEL=gpt-4o-mini
OPENAI_API_KEY=
CORS_ORIGINS=http://localhost:5173
REPORT_PDF_FILENAME=engedu-assessment-report.pdf
```

Production should use:

```text
APP_ENV=production
DATABASE_URL=postgresql+psycopg://user:password@localhost:5432/engedu
STORAGE_ROOT=/var/lib/engedu/storage
CORS_ORIGINS=https://your-domain.example
```

## 4. GitHub Workflow

### Branching

Recommended:

- `main`: stable, deployable.
- `develop`: integration branch if team wants one.
- `feature/phase-one-backend-foundation`
- `feature/phase-one-frontend-flow`
- `feature/phase-one-analysis`

### Pull Request Rules

Checklist:

- [ ] PR describes which docs were followed.
- [ ] Backend tests pass.
- [ ] Frontend build passes.
- [ ] API changes match `docs/api-endpoints.md`.
- [ ] Database changes match `docs/database-schema.md`.
- [ ] Prompt changes match `docs/prompts.md`.
- [ ] No secrets committed.

### Suggested GitHub Actions

Checks:

- Backend install.
- Backend tests.
- Backend lint/type check if configured.
- Frontend install.
- Frontend build.
- Frontend lint if configured.

## 5. VPS Deployment Workflow

High-level VPS deployment path:

```text
GitHub repository
  -> SSH to VPS
  -> pull latest release branch/tag
  -> install/update backend dependencies
  -> run database migrations
  -> build frontend
  -> restart backend service
  -> reload Nginx
  -> verify health endpoint
```

Recommended server layout:

```text
/opt/engedu/app              # repository checkout
/opt/engedu/app/backend      # backend code
/opt/engedu/app/frontend     # frontend code
/var/lib/engedu/storage      # audio and report storage
/var/log/engedu              # app logs
/etc/engedu/backend.env      # environment variables
```

## 6. Backend Service Recommendation

Use systemd for FastAPI production service.

Example service outline:

```ini
[Unit]
Description=EngEdu FastAPI Backend
After=network.target

[Service]
User=engedu
Group=engedu
WorkingDirectory=/opt/engedu/app/backend
EnvironmentFile=/etc/engedu/backend.env
ExecStart=/opt/engedu/app/backend/.venv/bin/uvicorn app.main:app --host 127.0.0.1 --port 8000
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Production may later use Gunicorn with Uvicorn workers.

## 7. Nginx Configuration Recommendations

Nginx should:

- Serve frontend static build.
- Reverse proxy `/api/` to FastAPI.
- Reverse proxy `/health` if desired.
- Enforce HTTPS.
- Limit upload size.
- Add security headers.

Example outline:

```nginx
server {
    listen 80;
    server_name engedu.example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name engedu.example.com;

    client_max_body_size 30M;

    root /opt/engedu/app/frontend/dist;
    index index.html;

    location / {
        try_files $uri /index.html;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:8000/api/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /health {
        proxy_pass http://127.0.0.1:8000/health;
    }

    add_header X-Content-Type-Options nosniff always;
    add_header X-Frame-Options DENY always;
    add_header Referrer-Policy strict-origin-when-cross-origin always;
}
```

## 8. Database Deployment

### Local

- SQLite file inside backend directory or configured local data directory.

### Production

- PostgreSQL recommended.
- Use dedicated database user.
- Use least privilege.
- Run Alembic migrations during deployment.

Checklist:

- [ ] Create production database.
- [ ] Create production DB user.
- [ ] Configure `DATABASE_URL`.
- [ ] Run `alembic upgrade head`.
- [ ] Verify reading passages and interview questions are seeded.

## 9. Storage Deployment

Checklist:

- [ ] Create persistent storage directory.
- [ ] Ensure backend user owns storage directory.
- [ ] Keep storage outside repository checkout.
- [ ] Back up storage directory.
- [ ] Do not serve raw storage directory publicly.

Recommended:

```bash
sudo mkdir -p /var/lib/engedu/storage/audio
sudo mkdir -p /var/lib/engedu/storage/reports
sudo chown -R engedu:engedu /var/lib/engedu/storage
```

## 10. Backup Strategy

Minimum production backup:

- PostgreSQL daily backup.
- Storage directory daily backup.
- Keep 7 daily backups and 4 weekly backups.
- Test restore procedure monthly.

Checklist:

- [ ] Back up database.
- [ ] Back up audio files.
- [ ] Back up PDF reports.
- [ ] Back up environment config securely.
- [ ] Do not store backups in the same disk only.
- [ ] Encrypt backups if possible.

## 11. Monitoring Strategy

Monitor:

- Backend health endpoint.
- HTTP 5xx rate.
- Disk usage, especially storage directory.
- Database availability.
- Processing failures.
- Transcription failures.
- LLM failures.
- PDF generation failures.

Checklist:

- [ ] Add `/health` endpoint.
- [ ] Add structured logs.
- [ ] Add log rotation.
- [ ] Add disk usage alerts.
- [ ] Add uptime monitoring.
- [ ] Add error reporting before public launch.

## 12. Recovery Procedures

### Backend service down

1. Check service status.
2. Check logs.
3. Restart service.
4. Verify `/health`.

```bash
sudo systemctl status engedu-backend
sudo journalctl -u engedu-backend -n 200
sudo systemctl restart engedu-backend
curl https://engedu.example.com/health
```

### Database failure

1. Stop backend.
2. Restore latest database backup.
3. Run migrations if needed.
4. Start backend.
5. Verify health and sample session lookup.

### Storage corruption/missing files

1. Identify missing storage path from database.
2. Restore affected files from backup.
3. Verify report downloads and audio references.
4. Avoid deleting database rows unless retention/deletion policy requires it.

### Failed assessment processing

1. Inspect `assessment_sessions.processing_error`.
2. Verify audio files exist.
3. Verify transcription/LLM configuration.
4. Use retry processing endpoint.
5. If retry fails, preserve submitted data and create developer issue.

## 13. Deployment Verification Checklist

After each deployment:

- [ ] `GET /health` returns ok.
- [ ] Frontend loads.
- [ ] API docs load in non-production or protected environment.
- [ ] Database migrations are current.
- [ ] Reading passages are present.
- [ ] Interview questions are present.
- [ ] Audio upload works with small test file.
- [ ] Report PDF generation works in test session.
- [ ] Logs do not contain secrets.
- [ ] API responses do not expose absolute paths.
