# BuildGuard AI - CI/CD Build Failure Notification System with AI-Powered Code Suggestions

Jenkins-based CSE Minor Project. When a Jenkins build fails the system **detects** the failed stage, **analyses** the console log,
asks **Claude** for root cause + fixed code/Jenkinsfile, **notifies** via Email / Slack / Teams, and **stores** everything for history & charts.

**Stack:** React + Tailwind + Recharts | FastAPI + SQLAlchemy + SQLite | Anthropic Claude API | Jenkins REST API & webhook

## 1. Prerequisites (Windows)
Python 3.10+ (tick "Add to PATH"), Node.js 18+ LTS, VS Code. Jenkins is optional - Demo Mode works without it.

## 2. Run (two VS Code terminals)
```powershell
# Terminal 1 - backend
cd backend
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
copy .env.example .env        # then edit SECRET_KEY etc.
uvicorn app.main:app --reload --port 8000

# Terminal 2 - frontend
cd frontend
npm install
npm run dev
```
(or double-click `start-backend.bat` / `start-frontend.bat`).
Open **http://localhost:5173** - API docs at http://localhost:8000/docs.

## 3. Demo mode (default, `DEMO_MODE=true`)
Seeds a project, a Jenkins job and ~22 builds (failures are processed by the real pipeline). Log in with
`admin@demo.dev / Admin@123` (admin) or `dev@demo.dev / Dev@12345`. In **Projects** use **Simulate failed build** to trigger a full
detect -> analyse -> AI -> notify cycle. Without keys, AI answers come from a labelled rule engine and notifications are labelled `simulated`.
Set `DEMO_MODE=false` for real use (the first registered user becomes admin).

## 4. Claude API
1. Create a key at https://console.anthropic.com -> put it in `backend/.env`: `ANTHROPIC_API_KEY=sk-ant-...`
2. Restart the backend. Settings page shows "AI engine: Claude". Model is `ANTHROPIC_MODEL` (default `claude-sonnet-5-5`).
The key lives only in the backend; logs are scrubbed of tokens/passwords before being sent. If Claude is unreachable the app falls back to the demo engine.

## 5. Jenkins integration
**A. Webhook (push, recommended)**
1. Set `JENKINS_TOKEN=<long random string>` in `backend/.env`, restart backend.
2. Jenkins -> Manage Credentials -> add *Secret text* with ID `buildguard-token` = that string.
3. Copy the `post { failure { ... } }` block from `Jenkinsfile.example` (also shown in Settings) into your Jenkinsfile.
   If Jenkins runs in Docker use `http://host.docker.internal:8000`; on another machine use your laptop IP.
4. In the app: Projects -> create project -> **Connect job** using the exact Jenkins job name (`folder/job` for folders).

**B. Pull (Sync button)**: Jenkins -> your user -> Configure -> *API Token* -> Add new. In Settings -> Jenkins enter URL, username, token, press
**Test connection**. In Projects press **Load from Jenkins** to pick jobs and **Sync** to import recent builds (status, branch, commit, console logs).
If a webhook arrives without a log, the backend downloads `consoleText` through this connection.

## 6. Notifications
* **Email:** fill `SMTP_*` in `.env` (Gmail: app password, host `smtp.gmail.com`, port 587). Recipient is set in Settings.
* **Slack:** create an Incoming Webhook, paste URL in Settings. **Teams:** create a Workflows/Incoming webhook, paste URL in Settings.
* Each message has project, job, build ID, failed stage, error, time, AI solution and dashboard link. History is under *Notifications*.

## 7. Feature map
Landing, login/register | Dashboard (real DB stats, charts) | Projects & Jenkins jobs | Build History (search/filter) |
Log Analysis (highlighted lines, timestamps, upload `.log/.txt`) | AI Code Suggestions (editable syntax-highlighted editor, copy, download, regenerate) |
AI Assistant (uses the build's log as context) | Notifications | Admin panel | Settings | dark/light theme.

Detected stages: dependency installation, compilation, testing, deployment, build, configuration (incl. Jenkinsfile syntax errors).

## 8. Security
bcrypt password hashing, JWT auth on all endpoints (admin-only routes enforced server-side), constant-time webhook token check,
Pydantic input validation, upload type/size/binary checks, integration secrets encrypted with Fernet (derived from `SECRET_KEY`) and never returned to the browser,
secrets never in frontend code. **AI code is never executed or pushed anywhere - it is shown for review, copy and download only.**
Before deploying beyond a demo: use a strong `SECRET_KEY`, HTTPS, `DEMO_MODE=false`, and consider Postgres + rate limiting.

## 9. Structure
```
backend/app/  main.py config.py database.py models.py security.py seed.py
  routes/     auth projects builds ai notifications webhooks admin
  services/   analyzer (log analysis) ai (Claude + demo) jenkins notify pipeline samples
frontend/src/ pages/ components/ api.js context.jsx App.jsx
```
DB tables: users, projects, repositories (Jenkins jobs), builds, build_logs, failures, ai_analyses, notifications, integrations, activity.

## 10. Troubleshooting
* `pip`/`npm` not found - reinstall Python/Node and tick "Add to PATH"; reopen VS Code.
* Login says Network error - backend not running on port 8000.
* PowerShell blocks venv activation - run `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`.
* Jenkins "HTTP 403" - wrong username/API token (use an API token, not your password).


## Cloud deployment
See [DEPLOY-TO-RAILWAY.md](DEPLOY-TO-RAILWAY.md) for the two-service Railway deployment setup.

## Install BuildGuard AI as an app (PWA)

The frontend includes a Progressive Web App manifest, app icons, and a service worker. Once the frontend is deployed on an HTTPS public domain:

- **Windows:** Open the public app URL in Microsoft Edge or Chrome. Use the install icon in the address bar, or browser menu → **Install BuildGuard AI** / **Install page as app**.
- **Android:** Open the public app URL in Chrome. Use the browser menu → **Install app** or **Add to Home screen**.
- **iPhone/iPad:** Open the public app URL in Safari → Share → **Add to Home Screen**.

The app opens in its own window after installation. A live deployment and HTTPS URL are required before installation is available. The service worker caches the app shell and static assets; API requests are not cached, so live dashboard data still requires a network connection.
