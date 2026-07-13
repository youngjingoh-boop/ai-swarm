<template>
  <div class="home-container">
    <!-- Top navigation -->
    <nav class="navbar">
      <div class="nav-brand">AI <span class="accent">Swarm</span></div>
      <div class="nav-links">
        <a :href="UPSTREAM_URL" target="_blank" rel="noopener" class="source-link">
          Source <span class="arrow">↗</span>
        </a>
      </div>
    </nav>

    <div class="main-content">
      <!-- Hero -->
      <section class="hero-section">
        <p class="kicker">
          <span class="live-dot" aria-hidden="true"></span>
          <span class="counter">{{ agentCountLabel }}</span> AI PEOPLE, READY TO LIVE OUT YOUR QUESTION
        </p>
        <h1 class="hero-title">Ask 1,000 AI people<br />what happens next.</h1>
        <p class="hero-sub">
          Upload a report, a plan, a story. AI Swarm reads it and generates a whole
          society of AI personas — customers, investors, skeptics, fans — then lets
          them live it out and shows you how it ends.
        </p>
        <button class="hero-cta" @click="scrollToUpload">Run a prediction →</button>

        <!-- Persona field: the swarm, made visible -->
        <div class="persona-field" aria-hidden="true">
          <div class="field-backdrop"></div>

          <!-- The question the swarm is answering -->
          <div class="question-card">
            <span class="q-label">YOUR QUESTION</span>
            <span class="q-text">"Will the new pricing land?"</span>
            <div class="consensus">
              <div class="consensus-bar">
                <span class="seg seg-a" style="width: 62%"></span>
                <span class="seg seg-b" style="width: 23%"></span>
                <span class="seg seg-c" style="width: 15%"></span>
              </div>
              <div class="consensus-legend">
                <span><i class="sw sw-a"></i>62% adopt</span>
                <span><i class="sw sw-b"></i>23% hesitate</span>
                <span><i class="sw sw-c"></i>15% churn</span>
              </div>
              <span class="consensus-caption">consensus after 6 simulated weeks</span>
            </div>
          </div>

          <!-- Floating persona chips -->
          <div
            v-for="(p, i) in personas"
            :key="p.n"
            class="persona-chip"
            :class="p.depth"
            :style="{ top: p.y, left: p.x, animationDelay: (i * 0.12) + 's', '--drift-delay': (i * 0.7) + 's' }"
          >
            <span class="chip-avatar">{{ p.i }}</span>
            <span class="chip-meta">
              <span class="chip-name">{{ p.n }}</span>
              <span class="chip-role">{{ p.r }}</span>
            </span>
            <span v-if="p.q" class="chip-bubble" :style="{ animationDelay: p.qd }">{{ p.q }}</span>
          </div>

          <div class="more-chip">+ 988 more, generated from your document</div>
        </div>
      </section>

      <!-- Meet your swarm -->
      <section class="meet-swarm">
        <p class="eyebrow">THE SWARM</p>
        <h2 class="section-title">Meet a few of the thousand.</h2>
        <div class="persona-grid">
          <article v-for="c in featured" :key="c.n" class="persona-card">
            <div class="card-head">
              <span class="card-avatar">{{ c.i }}</span>
              <div>
                <div class="card-name">{{ c.n }}, {{ c.age }}</div>
                <div class="card-role">{{ c.r }} · {{ c.mbti }}</div>
              </div>
            </div>
            <div class="card-traits">
              <span v-for="t in c.traits" :key="t" class="trait">{{ t }}</span>
            </div>
            <p class="card-quote">“{{ c.quote }}”</p>
          </article>
        </div>
        <p class="swarm-note">
          …and 997 more — each generated from <em>your</em> document, with demographics,
          motivations, and memory. After the run, you can interview any of them.
        </p>
      </section>

      <!-- How it works -->
      <section class="how-it-works">
        <p class="eyebrow">HOW IT WORKS</p>
        <div class="steps-grid">
          <div class="step">
            <span class="step-num">01</span>
            <h3 class="step-title">Upload your world</h3>
            <p class="step-desc">A market report, a business plan, a chapter — anything with people and stakes in it.</p>
          </div>
          <div class="step">
            <span class="step-num">02</span>
            <h3 class="step-title">The swarm comes alive</h3>
            <p class="step-desc">AI Swarm maps who matters, then generates up to 1,000 personas that think for themselves.</p>
          </div>
          <div class="step">
            <span class="step-num">03</span>
            <h3 class="step-title">Read the future</h3>
            <p class="step-desc">They interact for simulated weeks. You get the prediction report — and can question anyone in it.</p>
          </div>
        </div>
      </section>

      <!-- Upload card -->
      <section class="upload-section" ref="uploadSection">
        <p class="eyebrow">START</p>
        <h2 class="section-title">Put your question to the swarm.</h2>
        <div class="upload-card">
          <div
            class="drop-zone"
            :class="{ 'drag-over': isDragOver, 'has-files': files.length > 0 }"
            @dragover.prevent="handleDragOver"
            @dragleave.prevent="handleDragLeave"
            @drop.prevent="handleDrop"
            @click="triggerFileInput"
          >
            <input
              ref="fileInput"
              type="file"
              multiple
              accept=".pdf,.md,.txt"
              @change="handleFileSelect"
              style="display: none"
              :disabled="loading"
            />

            <div v-if="files.length === 0" class="drop-placeholder">
              <div class="upload-glyph">↑</div>
              <div class="drop-title">Drop files to begin</div>
              <div class="drop-hint">PDF, Markdown, or TXT — or click to browse</div>
            </div>

            <div v-else class="file-list">
              <div v-for="(file, index) in files" :key="index" class="file-chip">
                <span class="file-name">{{ file.name }}</span>
                <button @click.stop="removeFile(index)" class="remove-btn" aria-label="Remove file">×</button>
              </div>
            </div>
          </div>

          <textarea
            v-model="formData.simulationRequirement"
            class="requirement-input"
            placeholder='Describe what you want to predict — e.g. "How will the market react to this product over 6 months?"'
            rows="5"
            :disabled="loading"
          ></textarea>

          <p v-if="error" class="error-text">{{ error }}</p>

          <button
            class="cta-btn"
            @click="startSimulation"
            :disabled="!canSubmit || loading"
          >
            <span v-if="!loading">Run a prediction →</span>
            <span v-else>Starting…</span>
          </button>
        </div>
      </section>

      <!-- Past projects -->
      <section class="past-projects">
        <h2 class="section-heading">Past projects</h2>
        <HistoryDatabase />
      </section>
    </div>

    <AppFooter />
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import HistoryDatabase from '../components/HistoryDatabase.vue'
import AppFooter from '../components/AppFooter.vue'
import { UPSTREAM_URL } from '../branding.js'

const router = useRouter()

// 表单数据
const formData = ref({
  simulationRequirement: ''
})

// 文件列表
const files = ref([])

// 状态
const loading = ref(false)
const error = ref('')
const isDragOver = ref(false)

// 文件输入引用
const fileInput = ref(null)

// Hero: live agent counter
const agentCount = ref(0)
const agentCountLabel = computed(() => agentCount.value.toLocaleString('en-US'))

onMounted(() => {
  const target = 1000
  const reduce = window.matchMedia('(prefers-reduced-motion: reduce)').matches
  if (reduce) {
    agentCount.value = target
    return
  }
  const t0 = performance.now()
  const dur = 1600
  const tick = (t) => {
    const p = Math.min((t - t0) / dur, 1)
    agentCount.value = Math.round(target * (1 - Math.pow(1 - p, 3)))
    if (p < 1) requestAnimationFrame(tick)
  }
  requestAnimationFrame(tick)
})

// Persona field data — the visible slice of the swarm
const personas = [
  { n: 'Maya Chen', r: 'Product manager', i: 'MC', x: '6%', y: '16%', depth: 'near', q: '“I’d switch if onboarding takes under a day.”', qd: '1.2s' },
  { n: 'Derek Okafor', r: 'Skeptical CFO', i: 'DO', x: '74%', y: '10%', depth: 'near', q: '“Show me the payback period.”', qd: '4.2s' },
  { n: 'Aki Tanaka', r: 'Student', i: 'AT', x: '66%', y: '68%', depth: 'near', q: '“My whole group chat knows by tonight.”', qd: '7.2s' },
  { n: 'Rosa Alvarez', r: 'Business owner', i: 'RA', x: '2%', y: '62%', depth: 'mid' },
  { n: 'Ben Carter', r: 'Tech journalist', i: 'BC', x: '30%', y: '4%', depth: 'mid' },
  { n: 'Priya Nair', r: 'Retail investor', i: 'PN', x: '86%', y: '42%', depth: 'mid' },
  { n: 'Sam Whitmore', r: 'Longtime customer', i: 'SW', x: '22%', y: '76%', depth: 'mid' },
  { n: 'Ingrid Ström', r: 'Designer', i: 'IS', x: '48%', y: '84%', depth: 'far' },
  { n: 'Omar Haddad', r: 'Growth marketer', i: 'OH', x: '92%', y: '74%', depth: 'far' },
  { n: 'Nina Petrova', r: 'Community mod', i: 'NP', x: '14%', y: '38%', depth: 'far' },
  { n: 'Jae Park', r: 'Data analyst', i: 'JP', x: '58%', y: '30%', depth: 'far' },
  { n: 'Luca Romano', r: 'Early adopter', i: 'LR', x: '38%', y: '58%', depth: 'far' }
]

// Featured persona cards
const featured = [
  {
    n: 'Maya Chen', age: 34, r: 'PRODUCT MANAGER', mbti: 'ENFJ', i: 'MC',
    traits: ['pragmatic', 'time-poor', 'team-first'],
    quote: 'I don’t buy tools. I buy time. Convince me in the first week or lose me.'
  },
  {
    n: 'Derek Okafor', age: 51, r: 'CFO', mbti: 'ISTJ', i: 'DO',
    traits: ['risk-averse', 'numbers-first'],
    quote: 'Every pitch sounds great. Payback period or it didn’t happen.'
  },
  {
    n: 'Aki Tanaka', age: 22, r: 'STUDENT', mbti: 'ENFP', i: 'AT',
    traits: ['trend-driven', 'vocal online'],
    quote: 'If it’s cool, my whole group chat knows by tonight. If it’s not, they know that too.'
  }
]

// 计算属性:是否可以提交
const canSubmit = computed(() => {
  return formData.value.simulationRequirement.trim() !== '' && files.value.length > 0
})

// 触发文件选择
const triggerFileInput = () => {
  if (!loading.value) {
    fileInput.value?.click()
  }
}

// 处理文件选择
const handleFileSelect = (event) => {
  const selectedFiles = Array.from(event.target.files)
  addFiles(selectedFiles)
}

// 处理拖拽相关
const handleDragOver = (e) => {
  if (!loading.value) {
    isDragOver.value = true
  }
}

const handleDragLeave = (e) => {
  isDragOver.value = false
}

const handleDrop = (e) => {
  isDragOver.value = false
  if (loading.value) return

  const droppedFiles = Array.from(e.dataTransfer.files)
  addFiles(droppedFiles)
}

// 添加文件
const addFiles = (newFiles) => {
  const validFiles = newFiles.filter(file => {
    const ext = file.name.split('.').pop().toLowerCase()
    return ['pdf', 'md', 'txt'].includes(ext)
  })
  files.value.push(...validFiles)
}

// 移除文件
const removeFile = (index) => {
  files.value.splice(index, 1)
}

// Scroll to the upload card
const uploadSection = ref(null)
const scrollToUpload = () => {
  uploadSection.value?.scrollIntoView({ behavior: 'smooth', block: 'start' })
}

// 开始模拟 - 立即跳转，API调用在Process页面进行
const startSimulation = () => {
  if (!canSubmit.value || loading.value) return

  // 存储待上传的数据
  import('../store/pendingUpload.js').then(({ setPendingUpload }) => {
    setPendingUpload(files.value, formData.value.simulationRequirement)

    // 立即跳转到Process页面（使用特殊标识表示新建项目）
    router.push({
      name: 'Process',
      params: { projectId: 'new' }
    })
  })
}
</script>

<style scoped>
.home-container {
  min-height: 100vh;
  background: var(--surface);
  font-family: var(--font-sans);
  color: var(--ink);
}

/* Navbar */
.navbar {
  height: 64px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 40px;
  border-bottom: 1px solid var(--border-light);
}

.nav-brand {
  font-weight: 700;
  font-size: 1.1rem;
  letter-spacing: -0.01em;
  color: var(--ink);
}

.nav-brand .accent {
  color: var(--accent);
}

.source-link {
  color: var(--muted);
  text-decoration: none;
  font-size: 0.9rem;
  font-weight: 500;
  display: flex;
  align-items: center;
  gap: 6px;
  transition: color 0.2s;
}

.source-link:hover {
  color: var(--accent);
}

/* Main content */
.main-content {
  max-width: 1080px;
  margin: 0 auto;
  padding: 0 40px;
}

/* Shared section headings */
.eyebrow {
  font-family: var(--font-mono);
  font-size: 0.72rem;
  letter-spacing: 0.14em;
  color: var(--accent);
  text-align: center;
  margin-bottom: 12px;
}

.section-title {
  font-size: clamp(26px, 3.4vw, 34px);
  font-weight: 700;
  letter-spacing: -0.02em;
  text-align: center;
  color: var(--ink);
  margin-bottom: 40px;
}

/* Hero */
.hero-section {
  margin: 0 auto;
  padding: 96px 0 72px;
  text-align: center;
  position: relative;
}

.kicker {
  font-family: var(--font-mono);
  font-size: 0.75rem;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: var(--muted);
  margin-bottom: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.kicker .counter {
  color: var(--accent);
  font-weight: 700;
  min-width: 4ch;
  text-align: right;
}

.live-dot {
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: var(--accent);
  animation: pulse 2.2s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% { box-shadow: 0 0 0 0 rgba(99, 91, 255, 0.4); }
  50% { box-shadow: 0 0 0 6px rgba(99, 91, 255, 0); }
}

.hero-title {
  font-size: clamp(42px, 6.4vw, 72px);
  font-weight: 700;
  letter-spacing: -0.03em;
  line-height: 1.05;
  color: var(--ink);
  margin-bottom: 24px;
}

.hero-sub {
  font-size: 1.15rem;
  line-height: 1.65;
  color: var(--muted);
  max-width: 600px;
  margin: 0 auto 32px;
}

.hero-cta {
  background: var(--accent);
  color: #fff;
  border: none;
  border-radius: 8px;
  padding: 14px 28px;
  font-weight: 600;
  font-size: 1rem;
  font-family: var(--font-sans);
  cursor: pointer;
  transition: background 0.2s, transform 0.15s;
}

.hero-cta:hover {
  background: var(--accent-hover);
  transform: translateY(-1px);
}

/* ===== Persona field — the signature ===== */
.persona-field {
  position: relative;
  max-width: 920px;
  height: 460px;
  margin: 64px auto 0;
}

.field-backdrop {
  position: absolute;
  inset: 0;
  background-image: radial-gradient(rgba(99, 91, 255, 0.14) 1.5px, transparent 1.5px);
  background-size: 26px 26px;
  -webkit-mask-image: radial-gradient(ellipse 65% 60% at 50% 50%, #000 30%, transparent 75%);
  mask-image: radial-gradient(ellipse 65% 60% at 50% 50%, #000 30%, transparent 75%);
}

/* Question card at the center */
.question-card {
  position: absolute;
  left: 50%;
  top: 44%;
  transform: translate(-50%, -50%);
  width: 300px;
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 14px;
  padding: 20px;
  box-shadow: 0 12px 40px rgba(26, 31, 54, 0.1);
  z-index: 3;
  text-align: left;
  animation: chipIn 0.7s 0.5s both;
}

.q-label {
  display: block;
  font-family: var(--font-mono);
  font-size: 0.62rem;
  letter-spacing: 0.14em;
  color: var(--muted);
  margin-bottom: 6px;
}

.q-text {
  display: block;
  font-weight: 600;
  font-size: 1.02rem;
  letter-spacing: -0.01em;
  color: var(--ink);
  margin-bottom: 16px;
}

.consensus-bar {
  display: flex;
  height: 8px;
  border-radius: 4px;
  overflow: hidden;
  gap: 2px;
  margin-bottom: 10px;
}

.seg { height: 100%; border-radius: 2px; }
.seg-a { background: var(--accent); }
.seg-b { background: rgba(99, 91, 255, 0.35); }
.seg-c { background: var(--border); }

.consensus-legend {
  display: flex;
  gap: 12px;
  font-family: var(--font-mono);
  font-size: 0.62rem;
  color: var(--muted);
  margin-bottom: 8px;
}

.consensus-legend span { display: flex; align-items: center; gap: 4px; }
.sw { width: 7px; height: 7px; border-radius: 2px; display: inline-block; }
.sw-a { background: var(--accent); }
.sw-b { background: rgba(99, 91, 255, 0.35); }
.sw-c { background: var(--border); }

.consensus-caption {
  font-family: var(--font-mono);
  font-size: 0.6rem;
  letter-spacing: 0.06em;
  color: var(--muted);
  opacity: 0.75;
}

/* Persona chips */
.persona-chip {
  position: absolute;
  display: flex;
  align-items: center;
  gap: 8px;
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 999px;
  padding: 6px 14px 6px 6px;
  box-shadow: 0 4px 16px rgba(26, 31, 54, 0.07);
  white-space: nowrap;
  animation: chipIn 0.6s both, drift 7s var(--drift-delay, 0s) ease-in-out infinite alternate;
}

.persona-chip.near { z-index: 4; }
.persona-chip.mid { opacity: 0.82; transform: scale(0.9); z-index: 2; }
.persona-chip.far { opacity: 0.55; transform: scale(0.78); filter: blur(0.4px); z-index: 1; }

.chip-avatar {
  width: 34px;
  height: 34px;
  border-radius: 50%;
  background: rgba(99, 91, 255, 0.12);
  color: var(--accent);
  font-weight: 700;
  font-size: 0.72rem;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.chip-meta {
  display: flex;
  flex-direction: column;
  text-align: left;
  line-height: 1.25;
}

.chip-name {
  font-weight: 600;
  font-size: 0.8rem;
  color: var(--ink);
}

.chip-role {
  font-family: var(--font-mono);
  font-size: 0.62rem;
  color: var(--muted);
}

/* Speech bubbles: one persona speaks at a time */
.chip-bubble {
  position: absolute;
  bottom: calc(100% + 10px);
  left: 12px;
  background: var(--ink);
  color: #fff;
  font-size: 0.72rem;
  line-height: 1.4;
  padding: 8px 12px;
  border-radius: 10px;
  border-bottom-left-radius: 2px;
  max-width: 230px;
  white-space: normal;
  opacity: 0;
  animation: speak 9s infinite;
  pointer-events: none;
}

@keyframes speak {
  0%, 8% { opacity: 0; transform: translateY(4px); }
  12%, 30% { opacity: 1; transform: translateY(0); }
  36%, 100% { opacity: 0; transform: translateY(4px); }
}

.more-chip {
  position: absolute;
  bottom: -8px;
  left: 50%;
  transform: translateX(-50%);
  font-family: var(--font-mono);
  font-size: 0.68rem;
  letter-spacing: 0.04em;
  color: var(--muted);
  background: var(--surface-alt);
  border: 1px solid var(--border-light);
  border-radius: 999px;
  padding: 7px 16px;
  animation: chipIn 0.6s 1.6s both;
}

@keyframes chipIn {
  from { opacity: 0; transform: translateY(14px); }
  /* no `to` frame: elements settle at their own computed opacity/transform,
     so depth layers (.mid/.far) keep their reduced opacity after entry */
}

/* entry animation ends at default transform; depth scaling folded into drift */
.persona-chip.mid { animation: chipIn 0.6s both, driftMid 7s var(--drift-delay, 0s) ease-in-out infinite alternate; }
.persona-chip.far { animation: chipIn 0.6s both, driftFar 8s var(--drift-delay, 0s) ease-in-out infinite alternate; }

@keyframes drift {
  from { transform: translateY(0); }
  to { transform: translateY(-12px); }
}

@keyframes driftMid {
  from { transform: scale(0.9) translateY(0); }
  to { transform: scale(0.9) translateY(-10px); }
}

@keyframes driftFar {
  from { transform: scale(0.78) translateY(0); }
  to { transform: scale(0.78) translateY(-8px); }
}

/* ===== Meet your swarm ===== */
.meet-swarm {
  padding: 72px 0;
  border-top: 1px solid var(--border-light);
}

.persona-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
}

.persona-card {
  border: 1px solid var(--border);
  border-radius: 14px;
  padding: 24px;
  background: var(--surface);
  transition: transform 0.2s, box-shadow 0.2s;
}

.persona-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 12px 32px rgba(26, 31, 54, 0.08);
}

.card-head {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 16px;
}

.card-avatar {
  width: 44px;
  height: 44px;
  border-radius: 50%;
  background: rgba(99, 91, 255, 0.12);
  color: var(--accent);
  font-weight: 700;
  font-size: 0.85rem;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.card-name {
  font-weight: 700;
  font-size: 1rem;
  color: var(--ink);
  letter-spacing: -0.01em;
}

.card-role {
  font-family: var(--font-mono);
  font-size: 0.68rem;
  letter-spacing: 0.06em;
  color: var(--muted);
  margin-top: 2px;
}

.card-traits {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin-bottom: 14px;
}

.trait {
  font-family: var(--font-mono);
  font-size: 0.65rem;
  color: var(--muted);
  background: var(--surface-alt);
  border: 1px solid var(--border-light);
  border-radius: 999px;
  padding: 3px 10px;
}

.card-quote {
  font-size: 0.92rem;
  line-height: 1.55;
  color: var(--ink);
}

.swarm-note {
  text-align: center;
  color: var(--muted);
  font-size: 0.92rem;
  max-width: 560px;
  margin: 32px auto 0;
  line-height: 1.6;
}

.swarm-note em {
  color: var(--ink);
  font-style: italic;
}

/* ===== How it works ===== */
.how-it-works {
  padding: 72px 0;
  border-top: 1px solid var(--border-light);
}

.steps-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 40px;
  margin-top: 28px;
}

.step { text-align: left; }

.step-num {
  font-family: var(--font-mono);
  color: var(--accent);
  font-weight: 700;
  font-size: 0.85rem;
  display: block;
  margin-bottom: 10px;
}

.step-title {
  font-size: 1.05rem;
  font-weight: 700;
  letter-spacing: -0.01em;
  color: var(--ink);
  margin-bottom: 8px;
}

.step-desc {
  font-size: 0.9rem;
  line-height: 1.6;
  color: var(--muted);
}

/* ===== Upload card ===== */
.upload-section {
  padding: 72px 0 80px;
  border-top: 1px solid var(--border-light);
}

.upload-card {
  max-width: 720px;
  margin: 0 auto;
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 16px;
  padding: 40px;
  box-shadow: 0 1px 3px rgba(26, 31, 54, 0.06);
}

.drop-zone {
  border: 1.5px dashed var(--border);
  border-radius: 12px;
  background: var(--surface-alt);
  min-height: 160px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.2s;
  margin-bottom: 20px;
}

.drop-zone.has-files {
  align-items: flex-start;
}

.drop-zone:hover {
  border-color: var(--accent);
}

.drop-zone.drag-over {
  border-color: var(--accent);
  background: rgba(99, 91, 255, 0.06);
}

.drop-placeholder {
  text-align: center;
  padding: 24px;
}

.upload-glyph {
  width: 40px;
  height: 40px;
  border: 1px solid var(--border);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto 14px;
  color: var(--accent);
  font-size: 1.1rem;
}

.drop-title {
  font-weight: 600;
  font-size: 0.95rem;
  color: var(--ink);
  margin-bottom: 6px;
}

.drop-hint {
  font-size: 0.8rem;
  color: var(--muted);
}

.file-list {
  width: 100%;
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.file-chip {
  display: flex;
  align-items: center;
  background: var(--surface);
  padding: 10px 14px;
  border: 1px solid var(--border);
  border-radius: 8px;
  font-family: var(--font-mono);
  font-size: 0.85rem;
}

.file-name {
  flex: 1;
  margin-right: 10px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.remove-btn {
  background: none;
  border: none;
  cursor: pointer;
  font-size: 1.1rem;
  color: var(--muted);
  line-height: 1;
}

.remove-btn:hover {
  color: var(--accent);
}

.requirement-input {
  width: 100%;
  border: 1px solid var(--border);
  background: var(--surface-alt);
  border-radius: 10px;
  padding: 16px;
  font-family: var(--font-sans);
  font-size: 0.95rem;
  line-height: 1.6;
  color: var(--ink);
  resize: vertical;
  outline: none;
  margin-bottom: 20px;
  transition: border-color 0.2s;
}

.requirement-input:focus {
  border-color: var(--accent);
}

.requirement-input:disabled {
  cursor: not-allowed;
  color: var(--muted);
}

.error-text {
  color: #e5484d;
  font-size: 0.85rem;
  margin-bottom: 16px;
}

.cta-btn {
  width: 100%;
  background: var(--accent);
  color: #fff;
  border: none;
  border-radius: 8px;
  padding: 16px;
  font-weight: 600;
  font-size: 1rem;
  cursor: pointer;
  transition: background 0.2s;
}

.cta-btn:hover:not(:disabled) {
  background: var(--accent-hover);
}

.cta-btn:disabled {
  background: var(--border);
  color: var(--muted);
  cursor: not-allowed;
}

/* Past projects */
.past-projects {
  padding-bottom: 80px;
  border-top: 1px solid var(--border-light);
  padding-top: 60px;
}

.section-heading {
  font-size: 1.5rem;
  font-weight: 700;
  letter-spacing: -0.01em;
  color: var(--ink);
  margin-bottom: 24px;
  text-align: center;
}

/* Reduced motion: everything lands instantly, nothing drifts */
@media (prefers-reduced-motion: reduce) {
  .persona-chip,
  .question-card,
  .more-chip,
  .live-dot {
    animation: none !important;
  }
  .chip-bubble {
    animation: none !important;
    opacity: 0;
  }
  .persona-chip.mid { transform: scale(0.9); }
  .persona-chip.far { transform: scale(0.78); }
}

/* Responsive */
@media (max-width: 900px) {
  .persona-grid,
  .steps-grid {
    grid-template-columns: 1fr;
    gap: 16px;
  }

  .persona-field {
    height: 520px;
  }
}

@media (max-width: 768px) {
  .navbar {
    padding: 0 20px;
  }

  .main-content {
    padding: 0 20px;
  }

  .hero-section {
    padding: 64px 0 48px;
  }

  .upload-card {
    padding: 24px;
  }

  /* Persona field on small screens: hide the far layer, tighten */
  .persona-chip.far {
    display: none;
  }

  .chip-bubble {
    display: none;
  }

  .question-card {
    width: 260px;
  }

  .kicker {
    flex-wrap: wrap;
    padding: 0 8px;
  }
}
</style>
