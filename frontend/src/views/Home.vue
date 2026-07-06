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
        <p class="kicker">SWARM INTELLIGENCE ENGINE</p>
        <h1 class="hero-title">Rehearse the future before it happens.</h1>
        <p class="hero-sub">
          Upload any report. AI Swarm builds a parallel world of AI agents from it,
          runs the future forward, and hands you the prediction.
        </p>

        <!-- Swarm motif -->
        <div class="swarm-motif" aria-hidden="true">
          <span class="dot dot-lg" style="top: 40%; left: 46%; animation-delay: 0s;"></span>
          <span class="dot dot-md" style="top: 20%; left: 30%; animation-delay: 0.4s;"></span>
          <span class="dot dot-sm" style="top: 15%; left: 60%; animation-delay: 0.8s;"></span>
          <span class="dot dot-xs" style="top: 55%; left: 20%; animation-delay: 1.2s;"></span>
          <span class="dot dot-md" style="top: 65%; left: 62%; animation-delay: 1.6s;"></span>
          <span class="dot dot-lg" style="top: 30%; left: 70%; animation-delay: 2s;"></span>
          <span class="dot dot-sm" style="top: 75%; left: 40%; animation-delay: 2.4s;"></span>
          <span class="dot dot-xs" style="top: 10%; left: 45%; animation-delay: 2.8s;"></span>
          <span class="dot dot-md" style="top: 50%; left: 12%; animation-delay: 3.2s;"></span>
          <span class="dot dot-sm" style="top: 68%; left: 78%; animation-delay: 3.6s;"></span>
          <span class="dot dot-xs" style="top: 35%; left: 15%; animation-delay: 4s;"></span>
          <span class="dot dot-lg" style="top: 58%; left: 52%; animation-delay: 4.4s;"></span>
        </div>

        <button class="scroll-down-btn" @click="scrollToBottom" aria-label="Scroll down">
          ↓
        </button>
      </section>

      <!-- Upload card -->
      <section class="upload-section">
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

      <!-- How it works -->
      <section class="how-it-works">
        <div class="step">
          <span class="step-num">01</span>
          <span class="step-label">Upload seed</span>
        </div>
        <div class="step">
          <span class="step-num">02</span>
          <span class="step-label">Simulate the swarm</span>
        </div>
        <div class="step">
          <span class="step-num">03</span>
          <span class="step-label">Read the future</span>
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
import { ref, computed } from 'vue'
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

// 滚动到底部
const scrollToBottom = () => {
  window.scrollTo({
    top: document.body.scrollHeight,
    behavior: 'smooth'
  })
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

/* Hero */
.hero-section {
  max-width: 720px;
  margin: 0 auto;
  padding: 120px 0 80px;
  text-align: center;
  position: relative;
}

.kicker {
  font-family: var(--font-mono);
  font-size: 0.75rem;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: var(--accent);
  margin-bottom: 24px;
}

.hero-title {
  font-size: clamp(40px, 6vw, 64px);
  font-weight: 700;
  letter-spacing: -0.02em;
  line-height: 1.1;
  color: var(--ink);
  margin-bottom: 24px;
}

.hero-sub {
  font-size: 1.15rem;
  line-height: 1.6;
  color: var(--muted);
  max-width: 560px;
  margin: 0 auto;
}

/* Swarm motif */
.swarm-motif {
  position: relative;
  width: 100%;
  max-width: 360px;
  height: 320px;
  margin: 56px auto 0;
}

.dot {
  position: absolute;
  border-radius: 50%;
  background: var(--accent);
  animation: float 4.5s ease-in-out infinite alternate;
}

.dot-xs {
  width: 6px;
  height: 6px;
  opacity: 0.3;
}

.dot-sm {
  width: 10px;
  height: 10px;
  opacity: 0.5;
}

.dot-md {
  width: 16px;
  height: 16px;
  opacity: 0.7;
}

.dot-lg {
  width: 22px;
  height: 22px;
  opacity: 1;
}

@keyframes float {
  0% { transform: translateY(0); }
  100% { transform: translateY(-18px); }
}

.scroll-down-btn {
  width: 40px;
  height: 40px;
  border: 1px solid var(--border);
  border-radius: 50%;
  background: transparent;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  color: var(--accent);
  font-size: 1.1rem;
  margin: 40px auto 0;
  transition: all 0.2s;
}

.scroll-down-btn:hover {
  border-color: var(--accent);
  background: var(--surface-alt);
}

/* Upload card */
.upload-section {
  padding: 0 0 80px;
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

/* How it works */
.how-it-works {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 48px;
  padding: 40px 0;
  border-top: 1px solid var(--border-light);
  margin-bottom: 80px;
}

.step {
  display: flex;
  align-items: center;
  gap: 10px;
}

.step-num {
  font-family: var(--font-mono);
  color: var(--accent);
  font-weight: 700;
  font-size: 0.9rem;
}

.step-label {
  color: var(--ink);
  font-size: 0.95rem;
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

/* Responsive */
@media (max-width: 768px) {
  .navbar {
    padding: 0 20px;
  }

  .main-content {
    padding: 0 20px;
  }

  .hero-section {
    padding: 72px 0 48px;
  }

  .upload-card {
    padding: 24px;
  }

  .how-it-works {
    gap: 24px;
  }
}
</style>
