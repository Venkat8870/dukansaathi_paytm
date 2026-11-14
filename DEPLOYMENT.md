# Live demo deployment

This project uses a Next.js frontend and a FastAPI backend. Deploy the frontend
to Vercel and the backend to Render. The backend loads the bundled Excel
workbooks from `backend/` when it starts.

## 1. Keep secrets out of GitHub

- Revoke any Sarvam API key that has been shared or exposed, and create a
  replacement.
- Put the replacement only in `backend/.env` for local development.
- Do not add `.env` or `.env.local` files to Git. The root `.gitignore` excludes
  them; `.env.example` files contain placeholders only.
- On Render, enter the replacement key in the service's Environment settings.

## 2. Push the deployment setup

From the project root in PowerShell:

```powershell
git status --short
git add .gitignore DEPLOYMENT.md backend/app.py backend/requirements.txt backend/.env.example frontend/.env.example
git status --short
git commit -m "Prepare app for live deployment"
git push origin main
```

Check `git status --short` before committing. It must not list `backend/.env`,
`frontend/.env.local`, or the local `env/` folder.

## 3. Deploy the backend to Render

1. Create a **Web Service** in Render and connect the GitHub repository
   `Venkat8870/dukansaathi_paytm`.
2. Set **Root Directory** to `backend`.
3. Set **Build Command** to `pip install -r requirements.txt`.
4. Set **Start Command** to
   `uvicorn app:app --host 0.0.0.0 --port $PORT`.
5. Add these environment variables in Render:
   - `SARVAM_API_KEY`: your new Sarvam key
   - `SARVAM_STT_MODEL`: `saaras:v4`
   - `SARVAM_STT_LANGUAGE`: `unknown`
   - `SARVAM_STT_MODE`: `transcribe`
   - `SARVAM_TTS_MODEL`: `bulbul:v4-flash`
   - `SARVAM_TTS_SPEAKER`: `shubh_enhi_ads`
   - `SARVAM_TRANSLATION_MODEL`: `sarvam-translate:v1`
   - `FRONTEND_ORIGINS`: fill this in after deploying the frontend
6. Deploy. Copy the service URL, for example
   `https://your-api-name.onrender.com`.
7. Open `https://your-api-name.onrender.com/status`. The response should show
   both datasets connected with non-zero record counts.

## 4. Deploy the frontend to Vercel

1. Create a Vercel project and import the same GitHub repository.
2. Set **Root Directory** to `frontend`.
3. Add the environment variable
   `NEXT_PUBLIC_API_BASE_URL=https://your-api-name.onrender.com` using your
   actual Render service URL (no trailing slash).
4. Deploy and copy the Vercel production URL.
5. In Render, set `FRONTEND_ORIGINS` to that exact Vercel URL, for example
   `https://your-project.vercel.app`, then redeploy the backend.

If you add a custom domain to Vercel, add that exact origin to
`FRONTEND_ORIGINS` too. Multiple origins can be comma-separated.

## 5. Verify the live demo

- Open the Vercel URL over HTTPS and confirm both connection cards show loaded
  data.
- Try a typed question first, then allow microphone access and try a voice
  question. The browser requires HTTPS for microphone access; Vercel provides
  it automatically.
- The first request to a free backend may take longer if its host has been
  idle. Start it a few minutes before presenting.

For local development, copy `backend/.env.example` to `backend/.env`, add your
local key, and use the ignored `frontend/.env.local` with
`NEXT_PUBLIC_API_BASE_URL=http://localhost:8000`.
