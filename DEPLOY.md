# Deploy: ai-swarm

Split deploy — **frontend on Vercel**, **backend on Render** (persistent container, needed for
long-running simulations). Everything below is prepared; you just run the auth + deploy steps
I can't do from a non-interactive session.

Both platforms deploy from a **GitHub repo**, so step 0 is pushing this code up.

---

## 0. Push to GitHub (once)

```bash
cd /Users/youngjinkoh/Miro-Fish/MiroFish
git init && git add . && git commit -m "Prep ai-swarm split deploy"
gh repo create ai-swarm --private --source=. --push   # or create manually + git push
```

> `.env` is git-ignored — your keys are NOT pushed. You'll set them in each dashboard instead.

> After pushing the fork, set `SOURCE_REPO_URL` in `frontend/src/branding.js` to point at your
> fork's URL — the app footer's "Source" link reads from this constant.

---

## 1. Backend → Render (do this FIRST — you need its URL for step 2)

Files already prepared: `backend/Dockerfile`, `backend/.dockerignore`, `render.yaml`.

1. https://dashboard.render.com → **New → Blueprint** → pick the `ai-swarm` repo.
   Render reads `render.yaml` and creates the `ai-swarm-backend` web service.
2. When prompted, fill the two secret env vars (marked `sync:false`):
   - `LLM_API_KEY`  → your Groq key (from `.env`)
   - `ZEP_API_KEY`  → your Zep key (from `.env`)
3. Deploy. When live, copy the URL, e.g. `https://ai-swarm-backend.onrender.com`
4. Sanity check: visiting `<that-url>/health` should return `{"status":"ok",...}`

> ⚠️ **1000 agents needs muscle.** The `free` plan (512 MB) will OOM on a large run. Bump
> `plan:` in `render.yaml` to `standard`+ (or set it in the UI) before a real 1000-agent run.
> Also: Render's disk is ephemeral — uploaded seeds/sim data reset on redeploy (fine for a demo;
> attach a Render Disk if you need persistence).

---

## 2. Frontend → Vercel

File already prepared: `frontend/vercel.json` (Vite build + SPA rewrite).

```bash
npm i -g vercel                 # if not installed
cd /Users/youngjinkoh/Miro-Fish/MiroFish/frontend
vercel login                    # interactive — opens browser
vercel link                     # create/link project; name it: ai-swarm

# Point the frontend at the Render backend from step 1:
vercel env add VITE_API_BASE_URL production
#   paste: https://ai-swarm-backend.onrender.com   (NO trailing slash)

vercel --prod                   # build + deploy; prints your live https URL
```

The printed `https://ai-swarm.vercel.app` (or similar) is your shareable link.

### 2a. Attach the branded subdomain `aiswarm.unchurn.co`

This app deploys as its own Vercel project served from `/` — so it lives on a
**subdomain**, NOT a subpath of the main site (a subpath would require changes to
the unchurn.co project's routing + a Vite base-path rebuild; the subdomain avoids both).

1. Vercel → the `ai-swarm` project → **Settings → Domains → Add** → `aiswarm.unchurn.co`.
2. Vercel shows a DNS record to create. At the DNS host for `unchurn.co`, add:
   `CNAME  aiswarm  →  cname.vercel-dns.com`
3. Wait for propagation; Vercel auto-issues the TLS cert. Live at `https://aiswarm.unchurn.co`.

> The repo is already pushed to `https://github.com/youngjingoh-boop/ai-swarm` and
> `SOURCE_REPO_URL` in `frontend/src/branding.js` already points at it. **Make that repo
> public before real users hit the hosted app** — the footer "Source" link is your AGPL §13
> source offer and must resolve for them.

---

## 3. Verify end-to-end

1. Open the Vercel URL — landing page loads.
2. Open DevTools → Network, start a small run — API calls should hit
   `https://ai-swarm-backend.onrender.com/api/...` and return 200 (not localhost, not 404).
3. If calls fail: re-check `VITE_API_BASE_URL` (must be set for the **production** env, no trailing
   slash) and that `/health` on the backend is green.

---

## Notes / gotchas

- **Rate limits:** Groq's free tier will throw `429` under a 1000-agent load. Swap
  `LLM_API_KEY`/`LLM_BASE_URL`/`LLM_MODEL_NAME` (in Render's env) for a paid, high-throughput
  provider for real large runs.
- **Not yet build-tested locally:** the frontend production build is verified; the backend
  `Dockerfile` could not be built here (no Docker on this machine) — Render builds it on first deploy.
  If it fails there, the log will point at the offending step.
- **Backend URL first, always** — the frontend bakes `VITE_API_BASE_URL` in at build time, so if you
  change the backend URL later, re-run `vercel --prod`.
