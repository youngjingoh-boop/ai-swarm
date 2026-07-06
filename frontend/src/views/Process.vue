<template>
  <div class="process-page">
    <!-- Top navigation bar -->
    <nav class="navbar">
      <div class="nav-brand" @click="goHome">MIROFISH</div>

      <!-- Center step indicator -->
      <div class="nav-center">
        <div class="step-badge">STEP 01</div>
        <div class="step-name">World</div>
      </div>

      <div class="nav-status">
        <span class="status-dot" :class="statusClass"></span>
        <span class="status-text">{{ statusText }}</span>
      </div>
    </nav>

    <!-- Main content area -->
    <div class="main-content">
      <!-- Left: live knowledge graph -->
      <div class="left-panel" :class="{ 'full-screen': isFullScreen }">
        <div class="panel-header">
          <div class="header-left">
            <span class="header-deco">◆</span>
            <span class="header-title">Live Knowledge Graph</span>
          </div>
          <div class="header-right">
            <template v-if="graphData">
              <span class="stat-item">{{ graphData.node_count || graphData.nodes?.length || 0 }} Nodes</span>
              <span class="stat-divider">|</span>
              <span class="stat-item">{{ graphData.edge_count || graphData.edges?.length || 0 }} Edges</span>
              <span class="stat-divider">|</span>
            </template>
            <div class="action-buttons">
                <button class="action-btn" @click="refreshGraph" :disabled="graphLoading" title="Refresh graph">
                  <span class="icon-refresh" :class="{ 'spinning': graphLoading }">↻</span>
                </button>
                <button class="action-btn" @click="toggleFullScreen" :title="isFullScreen ? 'Exit full screen' : 'Full screen'">
                  <span class="icon-fullscreen">{{ isFullScreen ? '↙' : '↗' }}</span>
                </button>
            </div>
          </div>
        </div>

        <div class="graph-container" ref="graphContainer">
          <!-- Graph visualization (shown whenever data is available) -->
          <div v-if="graphData" class="graph-view">
            <svg ref="graphSvg" class="graph-svg"></svg>
            <!-- Building indicator -->
            <div v-if="currentPhase === 1" class="graph-building-hint">
              <span class="building-dot"></span>
              Live updating…
            </div>

            <!-- Node/edge detail panel -->
            <div v-if="selectedItem" class="detail-panel">
              <div class="detail-panel-header">
                <span class="detail-title">{{ selectedItem.type === 'node' ? 'Node Details' : 'Relationship' }}</span>
                <span v-if="selectedItem.type === 'node'" class="detail-badge" :style="{ background: selectedItem.color }">
                  {{ selectedItem.entityType }}
                </span>
                <button class="detail-close" @click="closeDetailPanel">×</button>
              </div>
              
              <!-- Node details -->
              <div v-if="selectedItem.type === 'node'" class="detail-content">
                <div class="detail-row">
                  <span class="detail-label">Name:</span>
                  <span class="detail-value highlight">{{ selectedItem.data.name }}</span>
                </div>
                <div class="detail-row">
                  <span class="detail-label">UUID:</span>
                  <span class="detail-value uuid">{{ selectedItem.data.uuid }}</span>
                </div>
                <div class="detail-row" v-if="selectedItem.data.created_at">
                  <span class="detail-label">Created:</span>
                  <span class="detail-value">{{ formatDate(selectedItem.data.created_at) }}</span>
                </div>
                
                <!-- Properties / Attributes -->
                <div class="detail-section" v-if="selectedItem.data.attributes && Object.keys(selectedItem.data.attributes).length > 0">
                  <span class="detail-label">Properties:</span>
                  <div class="properties-list">
                    <div v-for="(value, key) in selectedItem.data.attributes" :key="key" class="property-item">
                      <span class="property-key">{{ key }}:</span>
                      <span class="property-value">{{ value }}</span>
                    </div>
                  </div>
                </div>
                
                <!-- Summary -->
                <div class="detail-section" v-if="selectedItem.data.summary">
                  <span class="detail-label">Summary:</span>
                  <p class="detail-summary">{{ selectedItem.data.summary }}</p>
                </div>
                
                <!-- Labels -->
                <div class="detail-row" v-if="selectedItem.data.labels?.length">
                  <span class="detail-label">Labels:</span>
                  <div class="detail-labels">
                    <span v-for="label in selectedItem.data.labels" :key="label" class="label-tag">{{ label }}</span>
                  </div>
                </div>
              </div>
              
              <!-- Edge details -->
              <div v-else class="detail-content">
                <!-- Relationship display -->
                <div class="edge-relation">
                  <span class="edge-source">{{ selectedItem.data.source_name || selectedItem.data.source_node_name }}</span>
                  <span class="edge-arrow">→</span>
                  <span class="edge-type">{{ selectedItem.data.name || selectedItem.data.fact_type || 'RELATED_TO' }}</span>
                  <span class="edge-arrow">→</span>
                  <span class="edge-target">{{ selectedItem.data.target_name || selectedItem.data.target_node_name }}</span>
                </div>
                
                <div class="detail-subtitle">Relationship</div>
                
                <div class="detail-row">
                  <span class="detail-label">UUID:</span>
                  <span class="detail-value uuid">{{ selectedItem.data.uuid }}</span>
                </div>
                <div class="detail-row">
                  <span class="detail-label">Label:</span>
                  <span class="detail-value">{{ selectedItem.data.name || selectedItem.data.fact_type || 'RELATED_TO' }}</span>
                </div>
                <div class="detail-row" v-if="selectedItem.data.fact_type">
                  <span class="detail-label">Type:</span>
                  <span class="detail-value">{{ selectedItem.data.fact_type }}</span>
                </div>
                
                <!-- Fact -->
                <div class="detail-section" v-if="selectedItem.data.fact">
                  <span class="detail-label">Fact:</span>
                  <p class="detail-summary">{{ selectedItem.data.fact }}</p>
                </div>
                
                <!-- Episodes -->
                <div class="detail-section" v-if="selectedItem.data.episodes?.length">
                  <span class="detail-label">Episodes:</span>
                  <div class="episodes-list">
                    <span v-for="ep in selectedItem.data.episodes" :key="ep" class="episode-tag">{{ ep }}</span>
                  </div>
                </div>
                
                <div class="detail-row" v-if="selectedItem.data.created_at">
                  <span class="detail-label">Created:</span>
                  <span class="detail-value">{{ formatDate(selectedItem.data.created_at) }}</span>
                </div>
                <div class="detail-row" v-if="selectedItem.data.valid_at">
                  <span class="detail-label">Valid From:</span>
                  <span class="detail-value">{{ formatDate(selectedItem.data.valid_at) }}</span>
                </div>
                <div class="detail-row" v-if="selectedItem.data.invalid_at">
                  <span class="detail-label">Invalid At:</span>
                  <span class="detail-value">{{ formatDate(selectedItem.data.invalid_at) }}</span>
                </div>
                <div class="detail-row" v-if="selectedItem.data.expired_at">
                  <span class="detail-label">Expired At:</span>
                  <span class="detail-value">{{ formatDate(selectedItem.data.expired_at) }}</span>
                </div>
              </div>
            </div>
          </div>
          
          <!-- Loading state -->
          <div v-else-if="graphLoading" class="graph-loading">
            <div class="loading-animation">
              <div class="loading-ring"></div>
              <div class="loading-ring"></div>
              <div class="loading-ring"></div>
            </div>
            <p class="loading-text">Loading graph data…</p>
          </div>

          <!-- Waiting to build -->
          <div v-else-if="currentPhase < 1" class="graph-waiting">
            <div class="waiting-icon">
              <svg viewBox="0 0 100 100" class="network-icon">
                <circle cx="50" cy="20" r="8" fill="none" stroke="#000" stroke-width="1.5"/>
                <circle cx="20" cy="60" r="8" fill="none" stroke="#000" stroke-width="1.5"/>
                <circle cx="80" cy="60" r="8" fill="none" stroke="#000" stroke-width="1.5"/>
                <circle cx="50" cy="80" r="8" fill="none" stroke="#000" stroke-width="1.5"/>
                <line x1="50" y1="28" x2="25" y2="54" stroke="#000" stroke-width="1"/>
                <line x1="50" y1="28" x2="75" y2="54" stroke="#000" stroke-width="1"/>
                <line x1="28" y1="60" x2="72" y2="60" stroke="#000" stroke-width="1" stroke-dasharray="4"/>
                <line x1="50" y1="72" x2="26" y2="66" stroke="#000" stroke-width="1"/>
                <line x1="50" y1="72" x2="74" y2="66" stroke="#000" stroke-width="1"/>
              </svg>
            </div>
            <p class="waiting-text">Waiting for ontology generation</p>
            <p class="waiting-hint">Graph building will start automatically once generation completes</p>
          </div>

          <!-- Building but no data yet -->
          <div v-else-if="currentPhase === 1 && !graphData" class="graph-waiting">
            <div class="loading-animation">
              <div class="loading-ring"></div>
              <div class="loading-ring"></div>
              <div class="loading-ring"></div>
            </div>
            <p class="waiting-text">Building knowledge graph…</p>
            <p class="waiting-hint">Data will appear shortly…</p>
          </div>

          <!-- Error state -->
          <div v-else-if="error" class="graph-error">
            <span class="error-icon">⚠</span>
            <p>{{ error }}</p>
          </div>
        </div>

        <!-- Graph legend -->
        <div v-if="graphData" class="graph-legend">
          <div class="legend-item" v-for="type in entityTypes" :key="type.name">
            <span class="legend-dot" :style="{ background: type.color }"></span>
            <span class="legend-label">{{ type.name }}</span>
            <span class="legend-count">{{ type.count }}</span>
          </div>
        </div>
      </div>

      <!-- Right: build process details -->
      <div class="right-panel" :class="{ 'hidden': isFullScreen }">
        <div class="panel-header dark-header">
          <span class="header-icon">▣</span>
          <span class="header-title">Build Process</span>
        </div>

        <div class="process-content">
          <!-- Phase 1: Ontology Generation -->
          <div class="process-phase" :class="{ 'active': currentPhase === 0, 'completed': currentPhase > 0 }">
            <div class="phase-header">
              <span class="phase-num">01</span>
              <div class="phase-info">
                <div class="phase-title">Ontology Generation</div>
                <div class="phase-api">/api/graph/ontology/generate</div>
              </div>
              <span class="phase-status" :class="getPhaseStatusClass(0)">
                {{ getPhaseStatusText(0) }}
              </span>
            </div>

            <div class="phase-detail">
              <div class="detail-section">
                <div class="detail-label">Endpoint</div>
                <div class="detail-content">
                  After documents are uploaded, the LLM analyzes their content and automatically generates an ontology structure suited for opinion simulation (entity types + relation types).
                </div>
              </div>

              <!-- Ontology generation progress -->
              <div class="detail-section" v-if="ontologyProgress && currentPhase === 0">
                <div class="detail-label">Generation Progress</div>
                <div class="ontology-progress">
                  <div class="progress-spinner"></div>
                  <span class="progress-text">{{ ontologyProgress.message }}</span>
                </div>
              </div>

              <!-- Generated ontology info -->
              <div class="detail-section" v-if="projectData?.ontology">
                <div class="detail-label">Generated Entity Types ({{ projectData.ontology.entity_types?.length || 0 }})</div>
                <div class="entity-tags">
                  <span
                    v-for="entity in projectData.ontology.entity_types"
                    :key="entity.name"
                    class="entity-tag"
                  >
                    {{ entity.name }}
                  </span>
                </div>
              </div>

              <div class="detail-section" v-if="projectData?.ontology">
                <div class="detail-label">Generated Relation Types ({{ projectData.ontology.relation_types?.length || 0 }})</div>
                <div class="relation-list">
                  <div
                    v-for="(rel, idx) in projectData.ontology.relation_types?.slice(0, 5) || []"
                    :key="idx"
                    class="relation-item"
                  >
                    <span class="rel-source">{{ rel.source_type }}</span>
                    <span class="rel-arrow">→</span>
                    <span class="rel-name">{{ rel.name }}</span>
                    <span class="rel-arrow">→</span>
                    <span class="rel-target">{{ rel.target_type }}</span>
                  </div>
                  <div v-if="(projectData.ontology.relation_types?.length || 0) > 5" class="relation-more">
                    +{{ projectData.ontology.relation_types.length - 5 }} more relations…
                  </div>
                </div>
              </div>

              <!-- Waiting state -->
              <div class="detail-section waiting-state" v-if="!projectData?.ontology && currentPhase === 0 && !ontologyProgress">
                <div class="waiting-hint">Waiting for ontology generation…</div>
              </div>
            </div>
          </div>

          <!-- Phase 2: Graph Building -->
          <div class="process-phase" :class="{ 'active': currentPhase === 1, 'completed': currentPhase > 1 }">
            <div class="phase-header">
              <span class="phase-num">02</span>
              <div class="phase-info">
                <div class="phase-title">Graph Building</div>
                <div class="phase-api">/api/graph/build</div>
              </div>
              <span class="phase-status" :class="getPhaseStatusClass(1)">
                {{ getPhaseStatusText(1) }}
              </span>
            </div>

            <div class="phase-detail">
              <div class="detail-section">
                <div class="detail-label">Endpoint</div>
                <div class="detail-content">
                  Based on the generated ontology, documents are chunked and sent to the Zep API to build the knowledge graph and extract entities and relations.
                </div>
              </div>

              <!-- Waiting for ontology to finish -->
              <div class="detail-section waiting-state" v-if="currentPhase < 1">
                <div class="waiting-hint">Waiting for ontology generation to finish…</div>
              </div>

              <!-- Build progress -->
              <div class="detail-section" v-if="buildProgress && currentPhase >= 1">
                <div class="detail-label">Build Progress</div>
                <div class="progress-bar">
                  <div class="progress-fill" :style="{ width: buildProgress.progress + '%' }"></div>
                </div>
                <div class="progress-info">
                  <span class="progress-message">{{ buildProgress.message }}</span>
                  <span class="progress-percent">{{ buildProgress.progress }}%</span>
                </div>
              </div>

              <div class="detail-section" v-if="graphData">
                <div class="detail-label">Build Result</div>
                <div class="build-result">
                  <div class="result-item">
                    <span class="result-value">{{ graphData.node_count }}</span>
                    <span class="result-label">Entity Nodes</span>
                  </div>
                  <div class="result-item">
                    <span class="result-value">{{ graphData.edge_count }}</span>
                    <span class="result-label">Relation Edges</span>
                  </div>
                  <div class="result-item">
                    <span class="result-value">{{ entityTypes.length }}</span>
                    <span class="result-label">Entity Types</span>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Phase 3: Complete -->
          <div class="process-phase" :class="{ 'active': currentPhase === 2, 'completed': currentPhase > 2 }">
            <div class="phase-header">
              <span class="phase-num">03</span>
              <div class="phase-info">
                <div class="phase-title">Build Complete</div>
                <div class="phase-api">Ready for the next step</div>
              </div>
              <span class="phase-status" :class="getPhaseStatusClass(2)">
                {{ getPhaseStatusText(2) }}
              </span>
            </div>
          </div>

          <!-- Next step button -->
          <div class="next-step-section" v-if="currentPhase >= 2">
            <button class="next-step-btn" @click="goToNextStep" :disabled="currentPhase < 2">
              Continue to World setup
              <span class="btn-arrow">→</span>
            </button>
          </div>
        </div>

        <!-- Project info panel -->
        <div class="project-panel">
          <div class="project-header">
            <span class="project-icon">◇</span>
            <span class="project-title">Project Info</span>
          </div>
          <div class="project-details" v-if="projectData">
            <div class="project-item">
              <span class="item-label">Project Name</span>
              <span class="item-value">{{ projectData.name }}</span>
            </div>
            <div class="project-item">
              <span class="item-label">Project ID</span>
              <span class="item-value code">{{ projectData.project_id }}</span>
            </div>
            <div class="project-item" v-if="projectData.graph_id">
              <span class="item-label">Graph ID</span>
              <span class="item-value code">{{ projectData.graph_id }}</span>
            </div>
            <div class="project-item">
              <span class="item-label">Simulation Requirement</span>
              <span class="item-value">{{ projectData.simulation_requirement || '-' }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, watch, nextTick } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { generateOntology, getProject, buildGraph, getTaskStatus, getGraphData } from '../api/graph'
import { getPendingUpload, clearPendingUpload } from '../store/pendingUpload'
import * as d3 from 'd3'

const route = useRoute()
const router = useRouter()

// Current project ID (may change from 'new' to the actual ID)
const currentProjectId = ref(route.params.projectId)

// State
const loading = ref(true)
const graphLoading = ref(false)
const error = ref('')
const projectData = ref(null)
const graphData = ref(null)
const buildProgress = ref(null)
const ontologyProgress = ref(null) // Ontology generation progress
const currentPhase = ref(-1) // -1: uploading, 0: generating ontology, 1: building graph, 2: complete
const selectedItem = ref(null) // Selected node or edge
const isFullScreen = ref(false)

// DOM refs
const graphContainer = ref(null)
const graphSvg = ref(null)

// Polling timer
let pollTimer = null

// Computed properties
const statusClass = computed(() => {
  if (error.value) return 'error'
  if (currentPhase.value >= 2) return 'completed'
  return 'processing'
})

const statusText = computed(() => {
  if (error.value) return 'Build Failed'
  if (currentPhase.value >= 2) return 'Build Complete'
  if (currentPhase.value === 1) return 'Building Graph'
  if (currentPhase.value === 0) return 'Generating Ontology'
  return 'Initializing'
})

const entityTypes = computed(() => {
  if (!graphData.value?.nodes) return []
  
  const typeMap = {}
  const colors = ['#FF6B35', '#004E89', '#7B2D8E', '#1A936F', '#C5283D', '#E9724C']
  
  graphData.value.nodes.forEach(node => {
    const type = node.labels?.find(l => l !== 'Entity') || 'Entity'
    if (!typeMap[type]) {
      typeMap[type] = { name: type, count: 0, color: colors[Object.keys(typeMap).length % colors.length] }
    }
    typeMap[type].count++
  })
  
  return Object.values(typeMap)
})

// Methods
const goHome = () => {
  router.push('/')
}

const goToNextStep = () => {
  // TODO: Continue to World setup step
  alert('World setup is coming soon…')
}

const toggleFullScreen = () => {
  isFullScreen.value = !isFullScreen.value
  // Wait for transition to finish then re-render graph
  setTimeout(() => {
    renderGraph()
  }, 350)
}

// Close detail panel
const closeDetailPanel = () => {
  selectedItem.value = null
}

// Format date
const formatDate = (dateStr) => {
  if (!dateStr) return '-'
  try {
    const date = new Date(dateStr)
    return date.toLocaleString('en-US', {
      year: 'numeric',
      month: 'short',
      day: 'numeric',
      hour: '2-digit',
      minute: '2-digit'
    })
  } catch {
    return dateStr
  }
}

// Select node
const selectNode = (nodeData, color) => {
  selectedItem.value = {
    type: 'node',
    data: nodeData,
    color: color,
    entityType: nodeData.labels?.find(l => l !== 'Entity' && l !== 'Node') || 'Entity'
  }
}

// Select edge
const selectEdge = (edgeData) => {
  selectedItem.value = {
    type: 'edge',
    data: edgeData
  }
}

const getPhaseStatusClass = (phase) => {
  if (currentPhase.value > phase) return 'completed'
  if (currentPhase.value === phase) return 'active'
  return 'pending'
}

const getPhaseStatusText = (phase) => {
  if (currentPhase.value > phase) return 'Completed'
  if (currentPhase.value === phase) {
    if (phase === 1 && buildProgress.value) {
      return `${buildProgress.value.progress}%`
    }
    return 'In Progress'
  }
  return 'Waiting'
}

// Initialize - handle a new project or load an existing one
const initProject = async () => {
  const paramProjectId = route.params.projectId

  if (paramProjectId === 'new') {
    // New project: get pending upload data from the store
    await handleNewProject()
  } else {
    // Load existing project
    currentProjectId.value = paramProjectId
    await loadProject()
  }
}

// Handle new project - call the ontology/generate API
const handleNewProject = async () => {
  const pending = getPendingUpload()

  if (!pending.isPending || pending.files.length === 0) {
    error.value = 'No pending files found. Please return to the home page and try again.'
    loading.value = false
    return
  }

  try {
    loading.value = true
    currentPhase.value = 0 // Ontology generation phase
    ontologyProgress.value = { message: 'Uploading files and analyzing documents…' }

    // Build FormData
    const formDataObj = new FormData()
    pending.files.forEach(file => {
      formDataObj.append('files', file)
    })
    formDataObj.append('simulation_requirement', pending.simulationRequirement)

    // Call the ontology generation API
    const response = await generateOntology(formDataObj)

    if (response.success) {
      // Clear pending upload data
      clearPendingUpload()

      // Update project ID and data
      currentProjectId.value = response.data.project_id
      projectData.value = response.data

      // Update the URL (without reloading the page)
      router.replace({
        name: 'Process',
        params: { projectId: response.data.project_id }
      })

      ontologyProgress.value = null

      // Automatically start graph building
      await startBuildGraph()
    } else {
      error.value = response.error || 'Ontology generation failed'
    }
  } catch (err) {
    console.error('Handle new project error:', err)
    error.value = 'Project initialization failed: ' + (err.message || 'Unknown error')
  } finally {
    loading.value = false
  }
}

// Load existing project data
const loadProject = async () => {
  try {
    loading.value = true
    const response = await getProject(currentProjectId.value)

    if (response.success) {
      projectData.value = response.data
      updatePhaseByStatus(response.data.status)

      // Automatically start graph building
      if (response.data.status === 'ontology_generated' && !response.data.graph_id) {
        await startBuildGraph()
      }

      // Continue polling an in-progress build task
      if (response.data.status === 'graph_building' && response.data.graph_build_task_id) {
        currentPhase.value = 1
        startPollingTask(response.data.graph_build_task_id)
      }

      // Load a completed graph
      if (response.data.status === 'graph_completed' && response.data.graph_id) {
        currentPhase.value = 2
        await loadGraph(response.data.graph_id)
      }
    } else {
      error.value = response.error || 'Failed to load project'
    }
  } catch (err) {
    console.error('Load project error:', err)
    error.value = 'Failed to load project: ' + (err.message || 'Unknown error')
  } finally {
    loading.value = false
  }
}

const updatePhaseByStatus = (status) => {
  switch (status) {
    case 'created':
    case 'ontology_generated':
      currentPhase.value = 0
      break
    case 'graph_building':
      currentPhase.value = 1
      break
    case 'graph_completed':
      currentPhase.value = 2
      break
    case 'failed':
      error.value = projectData.value?.error || 'Processing failed'
      break
  }
}

// Start graph building
const startBuildGraph = async () => {
  try {
    currentPhase.value = 1
    // Set initial progress
    buildProgress.value = {
      progress: 0,
      message: 'Starting graph build…'
    }

    const response = await buildGraph({ project_id: currentProjectId.value })

    if (response.success) {
      buildProgress.value.message = 'Graph build task started…'

      // Save task_id for polling
      const taskId = response.data.task_id

      // Start graph data polling (independent of task status polling)
      startGraphPolling()

      // Start task status polling
      startPollingTask(taskId)
    } else {
      error.value = response.error || 'Failed to start graph build'
      buildProgress.value = null
    }
  } catch (err) {
    console.error('Build graph error:', err)
    error.value = 'Failed to start graph build: ' + (err.message || 'Unknown error')
    buildProgress.value = null
  }
}

// Graph data polling timer
let graphPollTimer = null

// Start graph data polling
const startGraphPolling = () => {
  // Fetch immediately once
  fetchGraphData()

  // Automatically fetch graph data every 10 seconds
  graphPollTimer = setInterval(async () => {
    await fetchGraphData()
  }, 10000)
}

// Manually refresh the graph
const refreshGraph = async () => {
  graphLoading.value = true
  await fetchGraphData()
  graphLoading.value = false
}

// Stop graph data polling
const stopGraphPolling = () => {
  if (graphPollTimer) {
    clearInterval(graphPollTimer)
    graphPollTimer = null
  }
}

// Fetch graph data
const fetchGraphData = async () => {
  try {
    // First fetch project info to get the graph_id
    const projectResponse = await getProject(currentProjectId.value)

    if (projectResponse.success && projectResponse.data.graph_id) {
      const graphId = projectResponse.data.graph_id
      projectData.value = projectResponse.data

      // Fetch graph data
      const graphResponse = await getGraphData(graphId)

      if (graphResponse.success && graphResponse.data) {
        const newData = graphResponse.data
        const newNodeCount = newData.node_count || newData.nodes?.length || 0
        const oldNodeCount = graphData.value?.node_count || graphData.value?.nodes?.length || 0

        console.log('Fetching graph data, nodes:', newNodeCount, 'edges:', newData.edge_count || newData.edges?.length || 0)

        // Re-render when the data has changed
        if (newNodeCount !== oldNodeCount || !graphData.value) {
          graphData.value = newData
          await nextTick()
          renderGraph()
        }
      }
    }
  } catch (err) {
    console.log('Graph data fetch:', err.message || 'not ready')
  }
}

// Poll task status
const startPollingTask = (taskId) => {
  // Query immediately once
  pollTaskStatus(taskId)

  // Then poll on an interval
  pollTimer = setInterval(() => {
    pollTaskStatus(taskId)
  }, 2000)
}

// Check task status
const pollTaskStatus = async (taskId) => {
  try {
    const response = await getTaskStatus(taskId)

    if (response.success) {
      const task = response.data

      // Update the progress display
      buildProgress.value = {
        progress: task.progress || 0,
        message: task.message || 'Processing…'
      }

      console.log('Task status:', task.status, 'Progress:', task.progress)

      if (task.status === 'completed') {
        console.log('✅ Graph build complete, loading full data…')

        stopPolling()
        stopGraphPolling()
        currentPhase.value = 2

        // Update the progress display to the completed state
        buildProgress.value = {
          progress: 100,
          message: 'Build complete, loading graph…'
        }

        // Reload project data to get the graph_id
        const projectResponse = await getProject(currentProjectId.value)
        if (projectResponse.success) {
          projectData.value = projectResponse.data

          // Finally load the full graph data
          if (projectResponse.data.graph_id) {
            console.log('📊 Loading full graph:', projectResponse.data.graph_id)
            await loadGraph(projectResponse.data.graph_id)
            console.log('✅ Graph loaded')
          }
        }

        // Clear the progress display
        buildProgress.value = null
      } else if (task.status === 'failed') {
        stopPolling()
        stopGraphPolling()
        error.value = 'Graph build failed: ' + (task.error || 'Unknown error')
        buildProgress.value = null
      }
    }
  } catch (err) {
    console.error('Poll task error:', err)
  }
}

const stopPolling = () => {
  if (pollTimer) {
    clearInterval(pollTimer)
    pollTimer = null
  }
}

// Load graph data
const loadGraph = async (graphId) => {
  try {
    graphLoading.value = true
    const response = await getGraphData(graphId)

    if (response.success) {
      graphData.value = response.data
      await nextTick()
      renderGraph()
    }
  } catch (err) {
    console.error('Load graph error:', err)
  } finally {
    graphLoading.value = false
  }
}

// Render graph (D3.js)
const renderGraph = () => {
  if (!graphSvg.value || !graphData.value) {
    console.log('Cannot render: svg or data missing')
    return
  }

  const container = graphContainer.value
  if (!container) {
    console.log('Cannot render: container missing')
    return
  }

  // Get container dimensions
  const rect = container.getBoundingClientRect()
  const width = rect.width || 800
  const height = (rect.height || 600) - 60

  if (width <= 0 || height <= 0) {
    console.log('Cannot render: invalid dimensions', width, height)
    return
  }

  console.log('Rendering graph:', width, 'x', height)

  const svg = d3.select(graphSvg.value)
    .attr('width', width)
    .attr('height', height)
    .attr('viewBox', `0 0 ${width} ${height}`)

  svg.selectAll('*').remove()

  // Process node data
  const nodesData = graphData.value.nodes || []
  const edgesData = graphData.value.edges || []

  if (nodesData.length === 0) {
    console.log('No nodes to render')
    // Show empty state
    svg.append('text')
      .attr('x', width / 2)
      .attr('y', height / 2)
      .attr('text-anchor', 'middle')
      .attr('fill', '#999')
      .text('Waiting for graph data…')
    return
  }

  // Build a node map for name lookups
  const nodeMap = {}
  nodesData.forEach(n => {
    nodeMap[n.uuid] = n
  })

  const nodes = nodesData.map(n => ({
    id: n.uuid,
    name: n.name || 'Unnamed',
    type: n.labels?.find(l => l !== 'Entity' && l !== 'Node') || 'Entity',
    rawData: n // Preserve the raw data
  }))

  // Build a node ID set to filter valid edges
  const nodeIds = new Set(nodes.map(n => n.id))

  const edges = edgesData
    .filter(e => nodeIds.has(e.source_node_uuid) && nodeIds.has(e.target_node_uuid))
    .map(e => ({
      source: e.source_node_uuid,
      target: e.target_node_uuid,
      type: e.fact_type || e.name || 'RELATED_TO',
      rawData: {
        ...e,
        source_name: nodeMap[e.source_node_uuid]?.name || 'Unknown',
        target_name: nodeMap[e.target_node_uuid]?.name || 'Unknown'
      }
    }))

  console.log('Nodes:', nodes.length, 'Edges:', edges.length)

  // Color mapping
  const types = [...new Set(nodes.map(n => n.type))]
  const colorScale = d3.scaleOrdinal()
    .domain(types)
    .range(['#FF6B35', '#004E89', '#7B2D8E', '#1A936F', '#C5283D', '#E9724C', '#2D3436', '#6C5CE7'])

  // Force-directed layout
  const simulation = d3.forceSimulation(nodes)
    .force('link', d3.forceLink(edges).id(d => d.id).distance(100).strength(0.5))
    .force('charge', d3.forceManyBody().strength(-300))
    .force('center', d3.forceCenter(width / 2, height / 2))
    .force('collision', d3.forceCollide().radius(40))
    .force('x', d3.forceX(width / 2).strength(0.05))
    .force('y', d3.forceY(height / 2).strength(0.05))

  // Add zoom functionality
  const g = svg.append('g')

  svg.call(d3.zoom()
    .extent([[0, 0], [width, height]])
    .scaleExtent([0.2, 4])
    .on('zoom', (event) => {
      g.attr('transform', event.transform)
    }))

  // Draw edges (including a clickable transparent wide line)
  const linkGroup = g.append('g')
    .attr('class', 'links')
    .selectAll('g')
    .data(edges)
    .enter()
    .append('g')
    .style('cursor', 'pointer')
    .on('click', (event, d) => {
      event.stopPropagation()
      selectEdge(d.rawData)
    })

  // Visible thin line
  const link = linkGroup.append('line')
    .attr('stroke', '#ccc')
    .attr('stroke-width', 1.5)
    .attr('stroke-opacity', 0.6)

  // Transparent wide line for click hit-testing
  linkGroup.append('line')
    .attr('stroke', 'transparent')
    .attr('stroke-width', 10)

  // Edge labels
  const linkLabel = g.append('g')
    .attr('class', 'link-labels')
    .selectAll('text')
    .data(edges)
    .enter()
    .append('text')
    .attr('font-size', '9px')
    .attr('fill', '#999')
    .attr('text-anchor', 'middle')
    .text(d => d.type.length > 15 ? d.type.substring(0, 12) + '...' : d.type)

  // Draw nodes
  const node = g.append('g')
    .attr('class', 'nodes')
    .selectAll('g')
    .data(nodes)
    .enter()
    .append('g')
    .style('cursor', 'pointer')
    .on('click', (event, d) => {
      event.stopPropagation()
      selectNode(d.rawData, colorScale(d.type))
    })
    .call(d3.drag()
      .on('start', dragstarted)
      .on('drag', dragged)
      .on('end', dragended))

  node.append('circle')
    .attr('r', 10)
    .attr('fill', d => colorScale(d.type))
    .attr('stroke', '#fff')
    .attr('stroke-width', 2)
    .attr('class', 'node-circle')

  node.append('text')
    .attr('dx', 14)
    .attr('dy', 4)
    .text(d => d.name?.substring(0, 12) || '')
    .attr('font-size', '11px')
    .attr('fill', '#333')
    .attr('font-family', 'JetBrains Mono, monospace')

  // Click on empty space to close the detail panel
  svg.on('click', () => {
    closeDetailPanel()
  })

  simulation.on('tick', () => {
    // Update the position of all edges (visible line and transparent click area)
    linkGroup.selectAll('line')
      .attr('x1', d => d.source.x)
      .attr('y1', d => d.source.y)
      .attr('x2', d => d.target.x)
      .attr('y2', d => d.target.y)

    // Update edge label positions
    linkLabel
      .attr('x', d => (d.source.x + d.target.x) / 2)
      .attr('y', d => (d.source.y + d.target.y) / 2 - 5)

    node.attr('transform', d => `translate(${d.x},${d.y})`)
  })

  function dragstarted(event) {
    if (!event.active) simulation.alphaTarget(0.3).restart()
    event.subject.fx = event.subject.x
    event.subject.fy = event.subject.y
  }

  function dragged(event) {
    event.subject.fx = event.x
    event.subject.fy = event.y
  }

  function dragended(event) {
    if (!event.active) simulation.alphaTarget(0)
    event.subject.fx = null
    event.subject.fy = null
  }
}

// Watch for graph data changes
watch(graphData, () => {
  if (graphData.value) {
    nextTick(() => renderGraph())
  }
})

// Lifecycle
onMounted(() => {
  initProject()
})

onUnmounted(() => {
  stopPolling()
  stopGraphPolling()
})
</script>

<style scoped>
.process-page {
  min-height: 100vh;
  background: var(--surface);
  font-family: var(--font-sans);
  overflow: hidden; /* Prevent body scroll in fullscreen */
}

/* Navigation bar */
.navbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 24px;
  height: 56px;
  background: var(--ink);
  color: var(--surface);
  z-index: 10;
  position: relative;
}

.nav-brand {
  font-size: 1rem;
  font-weight: 700;
  letter-spacing: 0.1em;
  cursor: pointer;
  transition: opacity 0.2s;
}

.nav-brand:hover {
  opacity: 0.8;
}

.nav-center {
  display: flex;
  align-items: center;
  gap: 12px;
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
}

.step-badge {
  background: var(--accent);
  color: var(--surface);
  padding: 2px 8px;
  font-size: 0.7rem;
  font-weight: 600;
  letter-spacing: 0.05em;
  border-radius: 2px;
  font-family: var(--font-mono);
}

.step-name {
  font-size: 0.85rem;
  letter-spacing: 0.05em;
  color: var(--surface);
}

.nav-status {
  display: flex;
  align-items: center;
}

.status-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: var(--muted);
  margin-right: 8px;
}

.status-dot.processing {
  background: var(--accent);
  animation: pulse 1.5s infinite;
}

.status-dot.completed {
  background: #1A936F;
}

.status-dot.error {
  background: #C5283D;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

.status-text {
  font-size: 0.75rem;
  color: var(--muted);
}

/* Main content area */
.main-content {
  display: flex;
  height: calc(100vh - 56px);
  position: relative;
}

/* Left panel - 50% default */
.left-panel {
  width: 50%;
  flex: none; /* Fixed width initially */
  display: flex;
  flex-direction: column;
  border-right: 1px solid var(--border-light);
  transition: width 0.35s cubic-bezier(0.4, 0, 0.2, 1);
  background: var(--surface);
  z-index: 5;
}

.left-panel.full-screen {
  width: 100%;
  border-right: none;
}

.panel-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 12px 24px;
  border-bottom: 1px solid var(--border-light);
  background: var(--surface);
  height: 50px;
}

.header-left {
  display: flex;
  align-items: center;
  gap: 8px;
}

.header-deco {
  color: var(--accent);
  font-size: 0.8rem;
}

.header-title {
  font-size: 0.85rem;
  font-weight: 600;
  letter-spacing: 0.05em;
}

.header-right {
  display: flex;
  align-items: center;
  gap: 16px;
  font-size: 0.75rem;
  color: var(--muted);
}

.stat-item {
  display: flex;
  align-items: center;
  gap: 4px;
}

.stat-val {
  font-weight: 600;
  color: var(--ink);
}

.stat-divider {
  color: var(--border-light);
}

.action-buttons {
    display: flex;
    align-items: center;
    gap: 8px;
}

.action-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 24px;
  height: 24px;
  background: transparent;
  border: 1px solid transparent;
  cursor: pointer;
  transition: all 0.2s;
  color: var(--muted);
  border-radius: 2px;
}

.action-btn:hover:not(:disabled) {
  background: var(--surface-alt);
  color: var(--ink);
}

.action-btn:disabled {
  opacity: 0.3;
  cursor: not-allowed;
}

.icon-refresh, .icon-fullscreen {
  font-size: 1rem;
  line-height: 1;
}

.icon-refresh.spinning {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

/* Graph container */
.graph-container {
  flex: 1;
  position: relative;
  overflow: hidden;
}

.graph-loading,
.graph-waiting,
.graph-error {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
}

.loading-animation {
  position: relative;
  width: 80px;
  height: 80px;
  margin: 0 auto 20px;
}

.loading-ring {
  position: absolute;
  border: 2px solid transparent;
  border-radius: 50%;
  animation: ring-rotate 1.5s linear infinite;
}

.loading-ring:nth-child(1) {
  width: 80px;
  height: 80px;
  border-top-color: var(--ink);
}

.loading-ring:nth-child(2) {
  width: 60px;
  height: 60px;
  top: 10px;
  left: 10px;
  border-right-color: var(--accent);
  animation-delay: 0.2s;
}

.loading-ring:nth-child(3) {
  width: 40px;
  height: 40px;
  top: 20px;
  left: 20px;
  border-bottom-color: var(--muted);
  animation-delay: 0.4s;
}

@keyframes ring-rotate {
  to { transform: rotate(360deg); }
}

.loading-text,
.waiting-text {
  font-size: 0.9rem;
  color: var(--ink);
  margin: 0 0 8px;
}

.waiting-hint {
  font-size: 0.8rem;
  color: var(--muted);
  margin: 0;
}

.waiting-icon {
  margin-bottom: 20px;
}

.network-icon {
  width: 100px;
  height: 100px;
  opacity: 0.6;
}

.graph-view {
  width: 100%;
  height: 100%;
  position: relative;
}

.graph-svg {
  width: 100%;
  height: 100%;
  display: block;
}

.graph-building-hint {
  position: absolute;
  bottom: 16px;
  left: 16px;
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 16px;
  background: rgba(99, 91, 255, 0.1);
  border: 1px solid var(--accent);
  font-size: 0.8rem;
  color: var(--accent);
}

.building-dot {
  width: 8px;
  height: 8px;
  background: var(--accent);
  border-radius: 50%;
  animation: pulse 1s infinite;
}

/* Node/edge detail panel */
.detail-panel {
  position: absolute;
  top: 16px;
  right: 16px;
  width: 320px;
  max-height: calc(100% - 32px);
  background: var(--surface);
  border: 1px solid var(--border-light);
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  display: flex;
  flex-direction: column;
  z-index: 100;
}

.detail-panel-header {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 12px 16px;
  background: var(--surface-alt);
  border-bottom: 1px solid var(--border-light);
}

.detail-title {
  font-size: 0.9rem;
  font-weight: 600;
  color: var(--ink);
}

.detail-badge {
  padding: 2px 10px;
  font-size: 0.75rem;
  color: var(--surface);
  border-radius: 2px;
}

.detail-close {
  margin-left: auto;
  width: 24px;
  height: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: none;
  border: none;
  font-size: 1.2rem;
  color: var(--muted);
  cursor: pointer;
  transition: color 0.2s;
}

.detail-close:hover {
  color: var(--ink);
}

.detail-content {
  padding: 16px;
  overflow-y: auto;
  flex: 1;
}

.detail-row {
  display: flex;
  align-items: flex-start;
  margin-bottom: 12px;
}

.detail-label {
  font-size: 0.8rem;
  color: var(--muted);
  min-width: 70px;
  flex-shrink: 0;
}

.detail-value {
  font-size: 0.85rem;
  color: var(--ink);
  word-break: break-word;
}

.detail-value.uuid {
  font-family: var(--font-mono);
  font-size: 0.75rem;
  color: var(--muted);
}

.detail-section {
  margin-bottom: 12px;
}

.detail-summary {
  margin: 8px 0 0 0;
  font-size: 0.85rem;
  color: var(--ink);
  line-height: 1.6;
  padding: 10px;
  background: var(--surface-alt);
  border-left: 3px solid var(--accent);
}

.detail-labels {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
}

.label-tag {
  padding: 2px 8px;
  font-size: 0.75rem;
  background: var(--surface-alt);
  border: 1px solid var(--border-light);
  color: var(--muted);
}

/* Edge detail relationship display */
.edge-relation {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 8px;
  margin-bottom: 16px;
  padding: 12px;
  background: var(--surface-alt);
  border: 1px solid var(--border-light);
}

.edge-source,
.edge-target {
  font-size: 0.85rem;
  font-weight: 500;
  color: var(--ink);
}

.edge-arrow {
  color: var(--muted);
}

.edge-type {
  padding: 2px 8px;
  font-size: 0.75rem;
  background: var(--accent);
  color: var(--surface);
}

.detail-value.highlight {
  font-weight: 600;
  color: var(--ink);
}

.detail-subtitle {
  font-size: 0.9rem;
  font-weight: 600;
  color: var(--ink);
  margin: 16px 0 12px 0;
  padding-bottom: 8px;
  border-bottom: 1px solid var(--border-light);
}

/* Properties list */
.properties-list {
  margin-top: 8px;
  padding: 10px;
  background: var(--surface-alt);
  border: 1px solid var(--border-light);
}

.property-item {
  display: flex;
  margin-bottom: 6px;
  font-size: 0.85rem;
}

.property-item:last-child {
  margin-bottom: 0;
}

.property-key {
  color: var(--muted);
  margin-right: 8px;
  font-family: var(--font-mono);
}

.property-value {
  color: var(--ink);
  word-break: break-word;
}

/* Episodes list */
.episodes-list {
  margin-top: 8px;
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.episode-tag {
  display: block;
  padding: 6px 10px;
  font-size: 0.75rem;
  font-family: var(--font-mono);
  background: var(--surface-alt);
  border: 1px solid var(--border-light);
  color: var(--muted);
  word-break: break-all;
}

.error-icon {
  font-size: 2rem;
  display: block;
  margin-bottom: 10px;
}

/* Graph legend */
.graph-legend {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
  padding: 12px 24px;
  border-top: 1px solid var(--border-light);
  background: var(--surface-alt);
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 0.75rem;
}

.legend-dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
}

.legend-label {
  color: var(--ink);
}

.legend-count {
  color: var(--muted);
}

/* Right panel - 50% default */
.right-panel {
  width: 50%;
  flex: none;
  display: flex;
  flex-direction: column;
  background: var(--surface);
  transition: width 0.35s cubic-bezier(0.4, 0, 0.2, 1), opacity 0.3s ease, transform 0.3s ease;
  overflow: hidden;
  opacity: 1;
}

.right-panel.hidden {
  width: 0;
  opacity: 0;
  transform: translateX(20px);
  pointer-events: none;
}

.right-panel .panel-header.dark-header {
  background: var(--ink);
  color: var(--surface);
  border-bottom: none;
}

.right-panel .header-icon {
  color: var(--accent);
  margin-right: 8px;
}

/* Process content */
.process-content {
  flex: 1;
  overflow-y: auto;
  padding: 24px;
}

/* Process phase */
.process-phase {
  margin-bottom: 24px;
  border: 1px solid var(--border-light);
  opacity: 0.5;
  transition: all 0.3s;
}

.process-phase.active,
.process-phase.completed {
  opacity: 1;
}

.process-phase.active {
  border-color: var(--accent);
}

.process-phase.completed {
  border-color: #1A936F;
}

.phase-header {
  display: flex;
  align-items: flex-start;
  gap: 16px;
  padding: 16px;
  background: var(--surface-alt);
  border-bottom: 1px solid var(--border-light);
}

.process-phase.active .phase-header {
  background: var(--surface-alt);
}

.process-phase.completed .phase-header {
  background: #F2FAF6;
}

.phase-num {
  font-size: 1.5rem;
  font-weight: 700;
  color: var(--border);
  line-height: 1;
  font-family: var(--font-mono);
}

.process-phase.active .phase-num {
  color: var(--accent);
}

.process-phase.completed .phase-num {
  color: #1A936F;
}

.phase-info {
  flex: 1;
}

.phase-title {
  font-size: 1rem;
  font-weight: 600;
  margin-bottom: 4px;
}

.phase-api {
  font-size: 0.75rem;
  color: var(--muted);
  font-family: var(--font-mono);
}

.phase-status {
  font-size: 0.75rem;
  padding: 4px 10px;
  background: var(--border-light);
  color: var(--muted);
}

.phase-status.active {
  background: var(--accent);
  color: var(--surface);
}

.phase-status.completed {
  background: #1A936F;
  color: var(--surface);
}

/* Phase detail */
.phase-detail {
  padding: 16px;
}

/* Entity tags */
.entity-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.entity-tag {
  font-size: 0.75rem;
  padding: 4px 10px;
  background: var(--surface-alt);
  border: 1px solid var(--border-light);
  color: var(--ink);
}

/* Relation list */
.relation-list {
  font-size: 0.8rem;
}

.relation-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 6px 0;
  border-bottom: 1px dashed var(--border-light);
}

.relation-item:last-child {
  border-bottom: none;
}

.rel-source,
.rel-target {
  color: var(--ink);
}

.rel-arrow {
  color: var(--border);
}

.rel-name {
  color: var(--accent);
  font-weight: 500;
}

.relation-more {
  padding-top: 8px;
  color: var(--muted);
  font-size: 0.75rem;
}

/* Ontology generation progress */
.ontology-progress {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px;
  background: var(--surface-alt);
  border: 1px solid rgba(99, 91, 255, 0.15);
}

.progress-spinner {
  width: 20px;
  height: 20px;
  border: 2px solid rgba(99, 91, 255, 0.15);
  border-top-color: var(--accent);
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

.progress-text {
  font-size: 0.85rem;
  color: var(--ink);
}

/* Waiting state */
.waiting-state {
  padding: 16px;
  background: var(--surface-alt);
  border: 1px dashed var(--border-light);
  text-align: center;
}

.waiting-hint {
  font-size: 0.85rem;
  color: var(--muted);
}

/* Progress bar */
.progress-bar {
  height: 6px;
  background: var(--border-light);
  margin-bottom: 8px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: var(--accent);
  transition: width 0.3s;
}

.progress-info {
  display: flex;
  justify-content: space-between;
  font-size: 0.75rem;
}

.progress-message {
  color: var(--muted);
}

.progress-percent {
  color: var(--accent);
  font-weight: 600;
}

/* Build result */
.build-result {
  display: flex;
  gap: 16px;
}

.result-item {
  flex: 1;
  text-align: center;
  padding: 12px;
  background: var(--surface-alt);
}

.result-value {
  display: block;
  font-size: 1.5rem;
  font-weight: 700;
  color: var(--ink);
  margin-bottom: 4px;
  font-family: var(--font-mono);
}

.result-label {
  font-size: 0.7rem;
  color: var(--muted);
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

/* Next step button */
.next-step-section {
  margin-top: 24px;
  padding-top: 24px;
  border-top: 1px solid var(--border-light);
}

.next-step-btn {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  padding: 16px;
  background: var(--ink);
  color: var(--surface);
  border: none;
  font-size: 1rem;
  font-weight: 500;
  letter-spacing: 0.05em;
  cursor: pointer;
  transition: all 0.2s;
}

.next-step-btn:hover:not(:disabled) {
  background: var(--accent-hover);
}

.next-step-btn:disabled {
  background: var(--border);
  cursor: not-allowed;
}

.btn-arrow {
  font-size: 1.2rem;
}

/* Project info panel */
.project-panel {
  border-top: 1px solid var(--border-light);
  background: var(--surface-alt);
}

.project-header {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 12px 24px;
  border-bottom: 1px solid var(--border-light);
}

.project-icon {
  color: var(--accent);
}

.project-title {
  font-size: 0.85rem;
  font-weight: 600;
}

.project-details {
  padding: 16px 24px;
}

.project-item {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  padding: 8px 0;
  border-bottom: 1px dashed var(--border-light);
  font-size: 0.8rem;
}

.project-item:last-child {
  border-bottom: none;
}

.item-label {
  color: var(--muted);
  flex-shrink: 0;
}

.item-value {
  color: var(--ink);
  text-align: right;
  max-width: 60%;
  word-break: break-all;
}

.item-value.code {
  font-family: var(--font-mono);
  font-size: 0.75rem;
  color: var(--muted);
}

/* Responsive */
@media (max-width: 1024px) {
  .main-content {
    flex-direction: column;
  }

  .left-panel {
    width: 100% !important;
    border-right: none;
    border-bottom: 1px solid var(--border-light);
    height: 50vh;
  }

  .right-panel {
    width: 100% !important;
    height: 50vh;
    opacity: 1 !important;
    transform: none !important;
  }

  .right-panel.hidden {
      display: none;
  }
}
</style>