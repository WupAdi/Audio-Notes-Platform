# Deployment Guide

This guide explains how to deploy the Audio Notes Platform for production, as required by the internship task.

## Architecture Checklist
To run this application in production, you will need to provision:
1. **Frontend**: Next.js App Router (Node.js environment)
2. **Backend API**: FastAPI (Python environment)
3. **Background Worker**: Celery Worker (Python environment)
4. **Database**: PostgreSQL
5. **Message Broker**: Redis
6. **Storage**: S3-compatible object storage (e.g., AWS S3, Cloudflare R2, Backblaze B2, or managed MinIO)

## Deployment using Vercel and Railway

This is the recommended path for a fast, free/low-cost setup.

### 1. Provision Infrastructure on Railway.app
1. Go to [Railway](https://railway.app/) and create an account.
2. Click **New Project** and add a **PostgreSQL** database.
3. Click **New** again and add a **Redis** instance.
4. Note down the connection strings (`DATABASE_URL` and `REDIS_URL`) provided by Railway.

### 2. Deploy Backend & Worker on Railway
1. In the same Railway project, click **New** -> **GitHub Repo** and select this repository.
2. Railway will detect the `Dockerfile` inside the `backend/` directory if you configure the root directory to be `/backend`.
3. Go to the new service's **Settings** -> **Build** and ensure the Root Directory is set to `/backend`.
4. Add the following **Environment Variables**:
   - `DATABASE_URL` (From Railway)
   - `REDIS_URL` (From Railway)
   - `GNANI_API_KEY`
   - `GEMINI_API_KEY`
   - `CORS_ORIGINS` (Set this to your Vercel frontend URL once deployed)
   - `STORAGE_*` variables (Point these to an AWS S3 or Cloudflare R2 bucket)
5. Under **Settings** -> **Deploy**, set the Start Command to:
   `alembic upgrade head && uvicorn app.main:app --host 0.0.0.0 --port $PORT`
6. **Deploy the Worker:** Duplicate the backend service, but change the Start Command to:
   `celery -A app.worker.celery_app worker --loglevel=INFO`

### 3. Deploy Frontend on Vercel
1. Go to [Vercel](https://vercel.com/) and import this GitHub repository.
2. Set the **Root Directory** to `frontend`.
3. In the Environment Variables section, add:
   - `NEXT_PUBLIC_API_URL` (Set this to the public URL Railway generated for your FastAPI service, e.g., `https://audio-notes-api.up.railway.app/api`)
   - `NEXT_PUBLIC_REPOSITORY_URL` (Your GitHub repo link)
4. Click **Deploy**.

### Final Verification
1. Visit your Vercel URL.
2. Upload an audio file to ensure CORS is configured properly and that the file flows through FastAPI -> Storage -> Redis -> Celery Worker -> Gnani/Gemini -> PostgreSQL -> Frontend.
