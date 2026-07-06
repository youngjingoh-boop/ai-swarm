# AI Swarm Rebrand Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rebrand MiroFish as **AI Swarm** ("Predicting future.") — unchurn.co-matched visual system, full English UI, English generated content, MiroFish credit + AGPL-3.0 compliance.

**Architecture:** Token overlay: global CSS variables + base typography in `App.vue`, a `branding.js` constants module and `AppFooter.vue` shared component; full template/style rewrite of the landing (`Home.vue`, script block preserved); in-place restyle + string translation of 13 internal components; English translation of backend LLM prompt templates (JSON schemas preserved verbatim).

**Tech Stack:** Vue 3 + Vite (frontend), Flask + Python (backend prompts). No new dependencies. No i18n framework.

**Spec:** `docs/superpowers/specs/2026-07-06-ai-swarm-rebrand-design.md`

## Global Constraints

- Product name: **AI Swarm** — tagline: **Predicting future.**
- Design tokens (exact): accent `#635bff`, accent-hover `#5249e6`, ink `#1a1f36`, muted `#6b7280`, border `#e2e4ea`, border-light `#edeef2`, surface `#ffffff`, surface-alt `#f7f8fa`.
- Fonts: Inter (base, already loaded in `index.html`), JetBrains Mono only for step numbers/metrics/labels.
- Step names everywhere: **Upload → World → Simulate → Report → Chat**.
- All user-visible strings English. Code comments MAY stay Chinese. Backend log messages MAY stay Chinese.
- Backend prompts: translate prose; **preserve JSON field names and structure verbatim** (`bio`, `persona`, `country`, etc.); every generation prompt gains an explicit English-output instruction.
- Never modify `<script>` logic in Vue files except string literals and imports explicitly listed.
- Credit footer on landing + main app shell: `Built on MiroFish · Open source under AGPL-3.0 · Source`.
- Upstream link: `https://github.com/666ghj/MiroFish`. LICENSE file untouched.
- No MiroFish fish logo or Shanda logos rendered in the UI.
- After every task: `cd frontend && npm run build` must print `✓ built`.
- CJK-check command (ugrep is the system grep): `awk '/<template>/,/<\/template>/' <file> | grep -cP '[\p{Han}]'` — must print `0` for completed frontend files.
- Working dir: `/Users/youngjinkoh/Miro-Fish/MiroFish`. Commit after every task with the message given in the task.
- Dev preview runs already (frontend :3000, backend :5001); use it for visual checks.

## Shared Translation Glossary (use consistently in every task)

| Chinese | English |
|---|---|
| 知识图谱 / 图谱 | knowledge graph / graph |
| 图谱构建 | Graph building (step label: **World**) |
| 环境搭建 | Environment setup (step label: **World**) |
| 模拟 / 推演 | simulation / prediction |
| 智能体 / Agent | agent |
| 人设 / 画像 | persona / profile |
| 实体 | entity |
| 轮 / 轮数 | round / rounds |
| 报告 | report |
| 采访 | interview |
| 问卷 | survey |
| 上传 / 启动 / 停止 / 暂停 / 继续 | Upload / Start / Stop / Pause / Resume |
| 生成中… / 构建中… / 加载中… | Generating… / Building… / Loading… |
| 历史项目 | Past projects |
| 启动引擎 | Run a prediction → |
| 预测万物 | Predicting future. |

---

### Task 1: Brand tokens, base typography, page identity

**Files:**
- Modify: `frontend/src/App.vue` (style block only)
- Modify: `frontend/index.html`
- Create: `frontend/src/branding.js`
- Modify: `package.json:2-4`, `frontend/package.json` (name/description only)

**Interfaces:**
- Produces: CSS variables `--accent --accent-hover --ink --muted --border --border-light --surface --surface-alt --font-sans --font-mono` (used by every later task); `branding.js` exports `APP_NAME`, `APP_TAGLINE`, `UPSTREAM_URL`, `SOURCE_REPO_URL`.

- [ ] **Step 1: Replace `App.vue` global style block** with:

```css
:root {
  --accent: #635bff;
  --accent-hover: #5249e6;
  --ink: #1a1f36;
  --muted: #6b7280;
  --border: #e2e4ea;
  --border-light: #edeef2;
  --surface: #ffffff;
  --surface-alt: #f7f8fa;
  --font-sans: 'Inter', -apple-system, BlinkMacSystemFont, 'Noto Sans SC', sans-serif;
  --font-mono: 'JetBrains Mono', monospace;
}
* { margin: 0; padding: 0; box-sizing: border-box; }
#app {
  font-family: var(--font-sans);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: var(--ink);
  background-color: var(--surface);
}
::-webkit-scrollbar { width: 8px; height: 8px; }
::-webkit-scrollbar-track { background: var(--surface-alt); }
::-webkit-scrollbar-thumb { background: #c2c6d0; border-radius: 4px; }
::-webkit-scrollbar-thumb:hover { background: var(--muted); }
button { font-family: inherit; }
a { color: var(--accent); }
```

Also translate the one Chinese comment-visible piece: keep template as-is (`<router-view />`).

- [ ] **Step 2: Create `frontend/src/branding.js`**:

```js
// Single source of truth for brand strings and links.
export const APP_NAME = 'AI Swarm'
export const APP_TAGLINE = 'Predicting future.'
export const UPSTREAM_URL = 'https://github.com/666ghj/MiroFish'
// Update to your own fork URL after pushing (AGPL §13 source offer).
export const SOURCE_REPO_URL = 'https://github.com/666ghj/MiroFish'
```

- [ ] **Step 3: Update `frontend/index.html`**: `lang="en"`; `<meta name="description" content="AI Swarm — upload any report, simulate a parallel world of AI agents, predict what happens next." />`; `<title>AI Swarm — Predicting future.</title>`. Keep the existing Google Fonts link (Inter already included).

- [ ] **Step 4: Rename packages.** Root `package.json`: `"name": "ai-swarm"`, `"description": "AI Swarm — Predicting future. A swarm-intelligence prediction engine (built on MiroFish)"`. `frontend/package.json`: `"name": "ai-swarm-frontend"`.

- [ ] **Step 5: Verify**: `cd frontend && npm run build` → `✓ built`. Browser tab at :3000 shows "AI Swarm — Predicting future."

- [ ] **Step 6: Commit**: `git add -A && git commit -m "feat: AI Swarm brand tokens, page identity, branding module"`

---

### Task 2: Shared footer component

**Files:**
- Create: `frontend/src/components/AppFooter.vue`

**Interfaces:**
- Consumes: `branding.js` exports from Task 1.
- Produces: `<AppFooter />` (no props) — imported by Home.vue (Task 3) and MainView.vue (Task 4).

- [ ] **Step 1: Create the component**:

```vue
<template>
  <footer class="app-footer">
    <span class="footer-brand">{{ APP_NAME }} — {{ APP_TAGLINE }}</span>
    <span class="footer-credit">
      Built on <a :href="UPSTREAM_URL" target="_blank" rel="noopener">MiroFish</a>
      · Open source under <a href="https://www.gnu.org/licenses/agpl-3.0.html" target="_blank" rel="noopener">AGPL-3.0</a>
      · <a :href="SOURCE_REPO_URL" target="_blank" rel="noopener">Source</a>
    </span>
  </footer>
</template>

<script setup>
import { APP_NAME, APP_TAGLINE, UPSTREAM_URL, SOURCE_REPO_URL } from '../branding'
</script>

<style scoped>
.app-footer {
  display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; gap: 8px;
  padding: 20px 40px; border-top: 1px solid var(--border-light);
  font-size: 13px; color: var(--muted);
}
.app-footer a { color: var(--muted); text-decoration: underline; }
.app-footer a:hover { color: var(--accent); }
.footer-brand { font-weight: 600; color: var(--ink); }
@media (max-width: 640px) { .app-footer { flex-direction: column; align-items: flex-start; } }
</style>
```

- [ ] **Step 2: Verify + commit**: build green → `git add -A && git commit -m "feat: shared AppFooter with MiroFish credit and AGPL source link"`

---

### Task 3: Landing page redesign (`Home.vue`)

**Files:**
- Modify: `frontend/src/views/Home.vue` (template + style rewrite; `<script setup>` kept except adding one import)

**Interfaces:**
- Consumes: existing script bindings — `fileInput` (ref), `files`, `formData.simulationRequirement`, `loading`, `error`, `isDragOver`, `triggerFileInput`, `handleFileSelect`, `handleDragOver`, `handleDragLeave`, `handleDrop`, `removeFile`, `startSimulation`, `scrollToBottom`, `<HistoryDatabase />`; `<AppFooter />` from Task 2.

- [ ] **Step 1: Read the current `<script setup>` block fully** to confirm the binding list above; add `import AppFooter from '../components/AppFooter.vue'`. Change NOTHING else in the script.

- [ ] **Step 2: Rewrite the template** to this structure (bind every handler/ref listed above exactly where its old counterpart was — upload zone keeps `@click="triggerFileInput"`, hidden `<input ref="fileInput" @change="handleFileSelect">`, drag handlers on the zone, CTA `@click="startSimulation"`):

  - Nav: left — wordmark `AI <span class="accent">Swarm</span>`; right — link "Source ↗" to `UPSTREAM_URL`.
  - Hero (single column, max-width 720px, generous whitespace): kicker line in mono/uppercase `SWARM INTELLIGENCE ENGINE`; H1 `Rehearse the future before it happens.`; sub `Upload any report. AI Swarm builds a parallel world of AI agents from it, runs the future forward, and hands you the prediction.`
  - Swarm motif: pure-CSS cluster of 12 `--accent` dots (varying size/opacity, subtle float animation) replacing all fish imagery. No `<img>` of the old logo anywhere.
  - Upload card: drop zone (dashed `--border`, `--surface-alt`; accent border when `isDragOver`), file chips with remove ×, textarea bound to `formData.simulationRequirement` with placeholder `Describe what you want to predict — e.g. "How will the market react to this product over 6 months?"`, error line `{{ error }}`, CTA button `Run a prediction →` (loading state: `Starting…`).
  - "How it works" strip: `01 Upload seed · 02 Simulate the swarm · 03 Read the future`, mono numbers in `--accent`.
  - `<HistoryDatabase />` section retitled `Past projects`.
  - `<AppFooter />` at the bottom.

- [ ] **Step 3: Rewrite the style block** with tokens: white background, `--ink` headings (Inter 600/700, H1 clamp(40px, 6vw, 64px), letter-spacing -0.02em), `--muted` body, accent CTA buttons (8px radius, hover `--accent-hover`), hairline `--border-light` section separators. Delete all black-brutalist rules.

- [ ] **Step 4: Verify**: build green; `awk '/<template>/,/<\/template>/' src/views/Home.vue | grep -cP '[\p{Han}]'` → `0`; screenshot :3000 — hero, upload zone, footer credit all render; drag-drop a file and confirm the chip appears (script untouched ⇒ behavior identical).

- [ ] **Step 5: Commit**: `git add -A && git commit -m "feat: AI Swarm landing page — unchurn-style hero, English copy, credit footer"`

---

### Task 4: App shell — `MainView.vue` + `Process.vue`

**Files:**
- Modify: `frontend/src/views/MainView.vue`, `frontend/src/views/Process.vue`

**Interfaces:**
- Consumes: tokens, `<AppFooter />`.
- Produces: step labels **Upload / World / Simulate / Report / Chat** used by the step indicator; later tasks assume these names in headers.

- [ ] **Step 1: MainView** — translate all 9 CJK lines; step indicator labels become Upload / World / Simulate / Report / Chat; restyle indicator (mono numbers, `--accent` for active, `--muted` inactive, `--border-light` connectors); add `<AppFooter />` under the main content area.
- [ ] **Step 2: Process.vue** (191 CJK lines) — translate every template string using the glossary; replace hardcoded colors with tokens; graph-status text in English (`Building knowledge graph…`, `Nodes`, `Edges`).
- [ ] **Step 3: Verify**: build green; CJK template check → `0` for both files; click through Home → main flow in preview, step bar reads Upload/World/Simulate/Report/Chat.
- [ ] **Step 4: Commit**: `git commit -am "feat: English app shell with AI Swarm step names and tokens"`

---

### Task 5: Step 1–2 components + GraphPanel

**Files:**
- Modify: `frontend/src/components/Step1GraphBuild.vue` (24 CJK lines), `frontend/src/components/Step2EnvSetup.vue` (210 CJK lines), `frontend/src/components/GraphPanel.vue` (91 CJK lines)

- [ ] **Step 1: Remove every `api-note` element** (10 occurrences across Step panels — lines like `<p class="api-note">POST /api/simulation/create</p>`) and their CSS rules. Users never see endpoints.
- [ ] **Step 2: Translate + retokenize Step1GraphBuild** (headers like 图谱构建→`Building your world`, buttons per glossary).
- [ ] **Step 3: Translate + retokenize Step2EnvSetup** — persona cards, timeline, hot-topics, round-config modal; statuses like 生成中→`Generating agents…`. Keep every `v-if`/binding untouched.
- [ ] **Step 4: Translate + retokenize GraphPanel** — node/edge detail labels (实体→Entity, 关系→Relationship), force-graph colors: nodes `--accent`, edges `#c2c6d0`.
- [ ] **Step 5: Verify**: build green; CJK template check → `0` × 3 files; `grep -rn "api-note" src/` → no matches.
- [ ] **Step 6: Commit**: `git commit -am "feat: English World steps, remove raw API labels, token restyle"`

---

### Task 6: Simulation components

**Files:**
- Modify: `frontend/src/components/Step3Simulation.vue` (74), `frontend/src/views/SimulationView.vue` (42), `frontend/src/views/SimulationRunView.vue` (43)

- [ ] **Step 1: Translate + retokenize all three** — run controls per glossary (Start/Pause/Stop/Resume), progress strings `Round {n} of {total}`, feed titles `Agents are interacting…`, platform labels stay Twitter/Reddit.
- [ ] **Step 2: Verify**: build green; CJK template check → `0` × 3.
- [ ] **Step 3: Commit**: `git commit -am "feat: English simulation screens with brand tokens"`

---

### Task 7: Report components

**Files:**
- Modify: `frontend/src/components/Step4Report.vue` (227 CJK — largest file, 5150 lines), `frontend/src/views/ReportView.vue` (15)

- [ ] **Step 1: Translate + retokenize Step4Report** — chapter/workflow labels (生成报告→`Generate report`, 章节→`Chapter`, 工具调用→`Tool call`), interview Q&A labels, loading states `Writing chapter {n}…`. Do NOT touch the markdown-rendering logic.
- [ ] **Step 2: Translate + retokenize ReportView.**
- [ ] **Step 3: Verify**: build green; CJK template check → `0` × 2.
- [ ] **Step 4: Commit**: `git commit -am "feat: English report screens"`

---

### Task 8: Interaction + history + remaining views

**Files:**
- Modify: `frontend/src/components/Step5Interaction.vue` (90), `frontend/src/components/HistoryDatabase.vue` (144), `frontend/src/views/InteractionView.vue` (15)

- [ ] **Step 1: Translate + retokenize Step5Interaction** — chat UI (`Ask the Report Agent anything…`), interview/survey labels, agent-picker text.
- [ ] **Step 2: Translate + retokenize HistoryDatabase** — section title `Past projects`, card statuses (`Completed`, `Running`, `Round {x}/{y}`), detail modal.
- [ ] **Step 3: Translate + retokenize InteractionView.**
- [ ] **Step 4: Full-frontend sweep**: `for f in src/App.vue src/views/*.vue src/components/*.vue; do echo "$(awk '/<template>/,/<\/template>/' $f | grep -cP '[\p{Han}]') $f"; done` → every count `0`. Then check script-block string literals: `grep -nP '["\x27`][^"\x27`]*[\p{Han}]' src/**/*.vue | grep -v '^\s*//'` and translate any user-visible hits (alerts, computed labels, placeholder text).
- [ ] **Step 5: Verify**: build green; click through all 5 steps in preview.
- [ ] **Step 6: Commit**: `git commit -am "feat: complete English frontend — chat, history, sweep audit"`

---

### Task 9: Backend prompts I — personas + ontology

**Files:**
- Modify: `backend/app/services/oasis_profile_generator.py` (`_get_system_prompt`, `_build_individual_persona_prompt`, `_build_group_persona_prompt`, fallback profile dicts ~lines 780–845, `COUNTRIES` list)
- Modify: `backend/app/services/ontology_generator.py` (prompt templates)

**Interfaces:**
- Produces: same function signatures, same JSON output fields — only prompt language changes.

- [ ] **Step 1: Translate all prompt prose to English.** Preserve numbered JSON field instructions with identical field names (e.g. `6. country: 国家（使用中文，如"中国"）` → `6. country: country name in English, e.g. "United States"`). Append to every system prompt: `Write ALL output text in English, regardless of the input language.`
- [ ] **Step 2: Translate fallback dict strings** (bios/personas like "Official account for…" already English; 中国 defaults → sample from the translated COUNTRIES list).
- [ ] **Step 3: Verify prompts render**: `cd backend && uv run python -c "from app.services.oasis_profile_generator import OasisProfileGenerator; g=OasisProfileGenerator.__new__(OasisProfileGenerator); print(g._get_system_prompt(True)[:300])"` → English text, no exception. Then `uv run python -c "import app; app.create_app()"` → starts clean.
- [ ] **Step 4: Commit**: `git commit -am "feat: English persona and ontology prompts (output language: English)"`

---

### Task 10: Backend prompts II — simulation config + run scripts

**Files:**
- Modify: `backend/app/services/simulation_config_generator.py` (initial posts / event / time-config prompt templates)
- Modify: `backend/scripts/run_twitter_simulation.py`, `run_reddit_simulation.py`, `run_parallel_simulation.py` (only strings that become agent-visible content; progress `print()`s may stay Chinese)

- [ ] **Step 1: Translate config-generation prompts** (same rules as Task 9: prose → English, JSON field names verbatim, add the English-output instruction).
- [ ] **Step 2: Audit scripts** for content-shaping strings (initial post templates, interview prompt wrappers): `grep -nP '[\p{Han}]' scripts/run_*.py | grep -v print` — translate content strings, leave prints.
- [ ] **Step 3: Verify**: `uv run python -c "import ast; [ast.parse(open(f).read()) for f in ['scripts/run_twitter_simulation.py','scripts/run_reddit_simulation.py','scripts/run_parallel_simulation.py','app/services/simulation_config_generator.py']]"` → no syntax errors; backend restarts clean in dev preview.
- [ ] **Step 4: Commit**: `git commit -am "feat: English simulation-config prompts and agent-visible content"`

---

### Task 11: Backend prompts III — report agent

**Files:**
- Modify: `backend/app/services/report_agent.py` (2571 lines — system prompts, tool descriptions, chapter templates, interview/chat prompts)

- [ ] **Step 1: Translate all prompt/tool-description prose**; keep tool *names* and JSON/function-call schemas identical; add `Write the report and all replies in English.` to the report + chat system prompts.
- [ ] **Step 2: Verify**: `uv run python -c "import ast; ast.parse(open('app/services/report_agent.py').read())"` → clean; backend restart clean; if free-tier limits allow, generate a tiny report against an existing simulation and confirm English output.
- [ ] **Step 3: Commit**: `git commit -am "feat: English report-agent prompts"`

---

### Task 12: Attribution, README, final verification

**Files:**
- Modify: `README.md` (prepend AI Swarm section), `DEPLOY_JASON.md` (rename references `jason-1000-agents` → `ai-swarm`)
- Verify: whole app end-to-end

- [ ] **Step 1: Prepend to `README.md`**:

```markdown
# AI Swarm — Predicting future.

Upload any report. AI Swarm builds a parallel world of AI agents from it, runs the
future forward, and hands you the prediction.

**Attribution:** AI Swarm is a rebranded fork of
[MiroFish](https://github.com/666ghj/MiroFish) by the MiroFish team (Shanda Group),
used under the [AGPL-3.0](./LICENSE) license. Full source of this modified version is
available via the Source link in the app footer. Original documentation follows below.

---
```

- [ ] **Step 2: Update deploy runbook naming** (`jason-1000-agents` → `ai-swarm` throughout `DEPLOY_JASON.md` and `render.yaml` service name → `ai-swarm-backend`; note in runbook: set `SOURCE_REPO_URL` in `frontend/src/branding.js` after pushing the fork).
- [ ] **Step 3: Final verification**: `npm run build` green; full CJK sweep from Task 8 Step 4 still all-zero; smoke test in preview — upload a small English seed, run graph build, confirm English entities/personas appear; screenshot landing + one internal step.
- [ ] **Step 4: Commit**: `git commit -am "docs: AI Swarm attribution, deploy runbook rename, final audit"`
