# Arche — Frontend (React + Vite)

Web UI for **Arche**, a secure multi-modal RAG system. Stack: React 19 · Vite · React Router.

This repo is **frontend only**. The API lives in a separate repo (`arche-backend`), deployed on Render.
This guide deploys the frontend on **Vercel (Hobby/free plan)**.

> Last checked against Vercel's docs: October 2026. Dashboard labels move around occasionally; the ideas stay the same.

---

## What you need

- A GitHub account (to hold this repo)
- A [Vercel](https://vercel.com) account (sign up with GitHub)
- Your deployed backend URL from Render, e.g. `https://arche-backend.onrender.com` (deploy the backend first, or at least know its name)

---

## How the pieces fit together

```
Browser ──► Vercel (this repo: static React app)
                │  fetch(VITE_API_URL + "/auth/login", ...)
                ▼
          Render (arche-backend)  ──►  MongoDB Atlas
```

The only setting the frontend needs is **`VITE_API_URL`** — the backend URL **including `/api`**.
Vite bakes it into the JavaScript at **build time**, so after changing it you must **redeploy** on Vercel.

---

## Step 1 — Push this repo to GitHub

```bash
cd arche-frontend
git init
git add .
git status            # make sure .env is NOT listed (it is gitignored)
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/<you>/arche-frontend.git
git push -u origin main
```

## Step 2 — Import the project into Vercel

1. Vercel Dashboard → **Add New… → Project**.
2. Under *Import Git Repository*, pick `arche-frontend` (grant Vercel access to the repo if asked).
3. On the configure screen, Vercel should auto-detect everything:

   | Field | Value |
   |---|---|
   | Framework Preset | **Vite** |
   | Root Directory | `./` (this repo is frontend only) |
   | Build Command | `npm run build` (default) |
   | Output Directory | `dist` (default) |
   | Install Command | default |

   Node version: `package.json` pins `22.x` (Vite 8 needs Node 20.19+ or 22.12+). Don't change it unless you know why.

4. Expand **Environment Variables** and add:

   | Name | Value |
   |---|---|
   | `VITE_API_URL` | `https://<your-backend>.onrender.com/api` |

   - It **must end with `/api`** and have **no trailing slash**.
   - Apply it to **Production** (and Preview if you want preview deployments to work).

5. Click **Deploy**. It takes about a minute.

## Step 3 — Tell the backend about your Vercel URL (CORS)

Vercel gives you a URL like `https://arche-frontend.vercel.app` (see the project's *Domains* section).

1. Render → your backend service → **Environment** → set `CORS_ORIGIN` to exactly that URL (scheme + host, **no trailing `/`**).
2. Save; Render redeploys the backend automatically.

Without this step the browser blocks API calls ("CORS blocked").

## Step 4 — Test

1. Open `https://<your-backend>.onrender.com/api/health` — wait for `{"status":"ok", ...}`.
   (Render's free instance sleeps when idle, so the first request can take up to ~1 minute.)
2. Open your Vercel URL and sign in. On a fresh backend the default login is `admin` / `admin`
   — **change it immediately** (see the backend README, Step 7).
3. Refresh a deep link such as `/admin` — it should load, not 404. (That's handled by `vercel.json`.)

## Optional — Google sign-in

Only if the backend has `GOOGLE_CLIENT_ID` set. In Google Cloud Console → *APIs & Services → Credentials → OAuth client (Web application)*,
add your Vercel URL (e.g. `https://arche-frontend.vercel.app`) under **Authorized JavaScript origins**.
The frontend needs no variable for this — it fetches the client ID from the backend.

## Optional — custom domain

Vercel project → **Settings → Domains** → add your domain and follow the DNS instructions.
Then add the new origin to the backend's `CORS_ORIGIN` (comma-separated list).

---

## Run locally

```bash
cp .env.example .env     # set VITE_API_URL=http://localhost:5000/api
npm install
npm run dev              # http://localhost:5173
```

Other scripts: `npm run build`, `npm run preview`, `npm run lint`.
The backend must be running locally (see `arche-backend`), with `CORS_ORIGIN` allowing `http://localhost:5173` (localhost is always allowed).

## What's in this repo (deployment-relevant files)

| File | Purpose |
|---|---|
| `vercel.json` | Rewrites every path to `index.html` so React Router deep links and refreshes work |
| `.env.example` | Documents `VITE_API_URL` (copy to `.env` locally; set in Vercel dashboard for production) |
| `package.json` → `engines.node` | Pins Node 22 on Vercel |

## Troubleshooting

| Symptom | Likely cause / fix |
|---|---|
| Login says "network error" / requests go to `localhost:5000` | `VITE_API_URL` not set on Vercel (the app falls back to localhost). Add it, then **Redeploy**. |
| Requests go to `https://xxx.vercel.app/...` or return 404 | `VITE_API_URL` is missing `https://` or the `/api` suffix. |
| Console: **CORS blocked** / `blocked by CORS policy` | Backend `CORS_ORIGIN` doesn't exactly match your Vercel URL (check `https`, no trailing `/`). Preview URLs differ from production. |
| Changed `VITE_API_URL` but nothing changed | It's applied at build time — Deployments → ⋯ → **Redeploy** (untick "use existing build cache" if needed). |
| 404 when refreshing `/admin`, `/chat`, … | `vercel.json` missing from the repo root of the deployed project. |
| Build fails with a Node version error | Keep `engines.node` at `22.x`, or set Node 22 in Vercel → Settings → Build & Deployment. |
| Very slow first login after idle | The Render free backend was asleep; wait up to ~1 minute and retry. |

## Security notes

- Everything in `VITE_*` variables is **public** (shipped to every browser). Never put secrets there.
- Don't commit `.env` (it's gitignored here).
- Change the default `admin/admin` account before sharing the URL.
