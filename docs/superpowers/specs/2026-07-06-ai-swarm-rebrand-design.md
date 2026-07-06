# AI Swarm — MiroFish Rebrand Design

**Date:** 2026-07-06
**Status:** Approved by user (Approach A)
**Goal:** Rebrand MiroFish as **AI Swarm** ("Predicting future.") for self-hosted use by the owner, clients, and friends. Full English UI + English generated content. Visual branding matched to www.unchurn.co. Source stays open under AGPL-3.0 with credit to MiroFish.

## Context

- Upstream: [MiroFish](https://github.com/666ghj/MiroFish) (AGPL-3.0), Vue 3 + Vite frontend (~20k lines, 14 components/views, styles scoped per component, no global stylesheet beyond `App.vue`), Flask backend whose LLM prompt templates are written in Chinese.
- License analysis (done earlier this session): rebrand + hosted commercial-ish use is permitted under AGPL-3.0 **provided** users of the hosted instance are offered the modified source (§13). Trademarks (MiroFish name/logo, Shanda logos) must not be reused as our brand.
- unchurn.co design tokens (extracted from live CSS): accent `#635bff`, ink `#1a1f36`, muted `#6b7280`, borders `#e2e4ea` / `#edeef2`, white surfaces, sans+mono pairing, minimal Stripe-like aesthetic, arrow CTAs.

## Decisions (user-confirmed)

1. **Name/tagline:** AI Swarm — "Predicting future." (conventional "AI" capitalization).
2. **Language:** English UI **and** English generated content (backend prompts translated).
3. **Approach A:** token overlay + full landing redesign + in-place restyle of internal screens. No full rebuild.
4. **Credit:** footer credit to MiroFish on every page + README attribution.

## Design

### 1. Brand system (global tokens)

In `frontend/src/App.vue` global style block, define CSS variables and base typography:

```css
:root {
  --accent: #635bff;        /* primary actions, links, highlights */
  --accent-hover: #5249e6;
  --ink: #1a1f36;           /* primary text */
  --muted: #6b7280;         /* secondary text */
  --border: #e2e4ea;
  --border-light: #edeef2;
  --surface: #ffffff;
  --surface-alt: #f7f8fa;
  --font-sans: 'Inter', -apple-system, 'Noto Sans SC', sans-serif;
  --font-mono: 'JetBrains Mono', monospace;  /* small labels/metrics only */
}
```

- Load Inter (Google Fonts link in `index.html`, replacing/joining existing font links).
- Base font becomes `var(--font-sans)`; mono reserved for step numbers, metrics, code-ish labels.
- Neutral scrollbars (gray thumb), no black brutalist accents.
- Buttons: solid `--accent` bg, white text, 8px radius, hover `--accent-hover`; secondary = white bg + `--border` + `--ink`. CTAs end with `→`.

### 2. Landing page — `frontend/src/views/Home.vue` (full redesign)

unchurn-style single column, generous whitespace:

- Nav: "AI Swarm" wordmark (Inter bold, `--ink`; "Swarm" or the dot motif in `--accent`) — right side: GitHub/source link.
- Hero: headline "Rehearse the future before it happens." Subline: one sentence (upload any report, AI Swarm builds a parallel world of agents and predicts what happens next). CTA button `Run a prediction →` (routes to existing main flow). Keep the existing upload entry behavior intact.
- Replace fish imagery/logo with a minimal CSS/SVG swarm-dot motif (cluster of `--accent` dots). No new binary assets.
- "How it works" strip: 3 quiet steps (Upload seed → Simulate the swarm → Read the future).
- Footer (see §6).
- All existing routing/upload logic in Home.vue preserved — this is a template/style/copy rewrite, not a logic rewrite.

### 3. Internal screens — restyle in place

Files: `MainView.vue`, `Process.vue`, `SimulationView.vue`, `SimulationRunView.vue`, `ReportView.vue`, `InteractionView.vue`, and `components/Step1GraphBuild.vue`, `Step2EnvSetup.vue`, `Step3Simulation.vue`, `Step4Report.vue`, `Step5Interaction.vue`, `GraphPanel.vue`, `HistoryDatabase.vue`.

- Replace hardcoded blacks/whites/borders with the tokens; align buttons/cards/inputs to the brand system. Layout structure, component logic, API calls, polling, and graph rendering untouched.
- UX simplifications:
  - Remove user-visible raw API endpoint labels (e.g. `POST /api/simulation/create` in Step panels).
  - Step names → **Upload → World → Simulate → Report → Chat**.
  - Status/progress messages in plain English (e.g. "Building your world…", "Agents are interacting…").

### 4. English UI (translation in place)

- Translate every user-visible Chinese string in the 14 Vue files. No i18n framework (single-language product — YAGNI).
- `index.html`: `<title>AI Swarm — Predicting future.</title>`, `lang="en"`, favicon can stay/simplify.
- Root `package.json` + `frontend/package.json`: name → `ai-swarm`, description in English.
- Comments in code may remain Chinese (invisible; low-value churn) — only user-visible strings are in scope.

### 5. English generated content (backend prompts)

Translate LLM prompt templates + add explicit "Respond in English" instructions in:

- `backend/app/services/oasis_profile_generator.py` (persona system prompts + individual/group prompt builders; fallback profile strings like "中国" defaults → English)
- `backend/app/services/ontology_generator.py`
- `backend/app/services/report_agent.py` (report generation + chat)
- `backend/app/services/simulation_config_generator.py` (initial posts, event config)
- `backend/scripts/run_twitter_simulation.py` / `run_reddit_simulation.py` / `run_parallel_simulation.py` — only where prompt/content text is shaped (progress prints may stay Chinese).

Rule: seed material may be any language; generated output must be English. Backend *log* messages stay Chinese (out of scope; not user-visible).

### 6. Credit + AGPL compliance

- Footer component (inline in Home + main layout): `Built on MiroFish · Open source under AGPL-3.0 · Source` — MiroFish links to `https://github.com/666ghj/MiroFish`; Source links to a `SOURCE_REPO_URL` placeholder constant (single spot to update after the user pushes their repo).
- `LICENSE` unchanged. Existing copyright headers preserved.
- `README.md` (ours, new at top or separate section): AI Swarm intro + attribution block: "AI Swarm is a rebranded fork of MiroFish by the MiroFish team / Shanda, used under AGPL-3.0."
- No MiroFish/Shanda logos or trademarks used as our branding (fish logo removed from UI; static assets stay in repo — they're part of the upstream source we redistribute).

### 7. Error handling / risk

- Backend prompt translation is the riskiest change (JSON-format prompts must keep exact field names like `bio`, `persona`, `country`). Translate prose, preserve JSON schema instructions verbatim in structure.
- Frontend restyle must not touch `<script>` blocks except where strings live.
- Rate limits (Groq free tier) unchanged — out of scope.

### 8. Verification

1. `npm run build` green after each phase.
2. Landing: preview screenshot vs unchurn aesthetic; no Chinese text visible; footer credit present.
3. Smoke pass of the 5-step flow against the live backend (dev server already running): upload a small English seed, run graph build, confirm English personas/output where feasible on free-tier limits.
4. Grep audit: no user-visible CJK strings remain in `frontend/src` templates; no raw API endpoint labels rendered.

## Out of scope

- Backend log/comment translation, new features, i18n framework, deploy execution (existing `DEPLOY_JASON.md` runbook covers hosting; rename to AI Swarm naming when user deploys), dependency license audit beyond what's noted.
