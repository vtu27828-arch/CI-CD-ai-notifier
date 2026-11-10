# BuildGuard AI — Railway deployment guide

This repository is prepared for two Railway services from the same GitHub repository.

## 1. Push the project to GitHub
Upload the contents of this folder to a new GitHub repository (do not upload `.env`, API keys, or database files).

## 2. Create the backend service
In Railway, create a project from the GitHub repository. Set the service **Root Directory** to `/backend`. Railway will use `backend/Dockerfile` and `backend/railway.toml`. Generate a public domain.

Set backend variables:
- `SECRET_KEY`: a long random secret (do not reuse the example)
- `DEMO_MODE`: `true` for initial demo, `false` for real use
- `FRONTEND_URL`: the frontend public URL after creating the frontend service
- `DATABASE_URL`: attach a Railway PostgreSQL database and use its connection URL. If omitted, SQLite is used and its data may be lost on redeploy.
- `ANTHROPIC_API_KEY`: optional; only if live Claude analysis is required
- `JENKINS_TOKEN`: optional; configure only when Jenkins webhook integration is ready

The backend health endpoint is `/api/health`.

## 3. Create the frontend service
Create a second service from the same GitHub repository. Set its **Root Directory** to `/frontend`. Railway will use `frontend/Dockerfile` and `frontend/railway.toml`. Set build variable `VITE_API_BASE_URL` to the backend public URL, without a trailing slash, for example `https://your-backend.up.railway.app`. Generate a public domain for the frontend.

## 4. Finish CORS configuration
Set the backend variable `FRONTEND_URL` to the frontend public URL (including `https://`), then redeploy the backend.

## 5. Verify
Open the frontend public URL. Check `https://your-backend.up.railway.app/api/health` returns `{"status":"ok"}`.

## Demo accounts
When `DEMO_MODE=true`, seeded demo accounts are created on an empty database:
- `admin@demo.dev` / `Admin@123`
- `dev@demo.dev` / `Dev@12345`

Change demo passwords and set `DEMO_MODE=false` before using this with real data. Never commit secrets.
