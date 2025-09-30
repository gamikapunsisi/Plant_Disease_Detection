<script setup>
import { ref } from 'vue'
import { Client } from '@gradio/client'

// Hugging Face Space endpoint (Gradio)
const SPACE_URL = 'https://gamika99-plant-disease.hf.space'
const PREDICT_ROUTE = '/predict_image'

// Mirror of backend class labels for quick reference in UI
const CLASS_NAMES = [
  'Pepper__bell___Bacterial_spot', 'Pepper__bell___healthy',
  'Potato___Early_blight', 'Potato___Late_blight', 'Potato___healthy',
  'Tomato__Bacterial_spot', 'Tomato__Early_blight', 'Tomato__Late_blight',
  'Tomato__Leaf_Mold', 'Tomato__Septoria_leaf_spot',
  'Tomato__Spider_mites__Two_spotted_spider_mite',
  'Tomato__Target_Spot', 'Tomato__Tomato_Yellow_Leaf_Curl_Virus',
  'Tomato__Tomato_mosaic_virus', 'Tomato__healthy'
]

const fileInput = ref(null)
const imageFile = ref(null)
const imagePreviewUrl = ref('')
const isLoading = ref(false)
const errorMessage = ref('')
const predictionText = ref('')
const top5 = ref([]) // Array of { label, prob }
const isDragActive = ref(false)
const predictedDisease = ref('')
const confidencePct = ref(0)
const isHealthy = ref(false)
const hasResult = ref(false)

function onFileChange(event) {
  errorMessage.value = ''
  predictionText.value = ''
  top5.value = []
  predictedDisease.value = ''
  confidencePct.value = 0
  isHealthy.value = false
  hasResult.value = false

  const files = event.target.files
  if (!files || files.length === 0) {
    imageFile.value = null
    imagePreviewUrl.value = ''
    return
  }
  const file = files[0]
  imageFile.value = file
  const url = URL.createObjectURL(file)
  imagePreviewUrl.value = url
}

function openFileDialog() {
  if (fileInput.value) fileInput.value.click()
}

function onDragOver() {
  isDragActive.value = true
}

function onDragLeave() {
  isDragActive.value = false
}

function onDrop(e) {
  isDragActive.value = false
  const files = e.dataTransfer?.files
  if (!files || files.length === 0) return
  const file = files[0]
  const fakeEvent = { target: { files: [file] } }
  onFileChange(fakeEvent)
}

async function analyze() {
  errorMessage.value = ''
  predictionText.value = ''
  top5.value = []

  if (!imageFile.value) {
    errorMessage.value = 'Please select an image first.'
    return
  }

  try {
    isLoading.value = true
    const client = await Client.connect(SPACE_URL)
    const result = await client.predict(PREDICT_ROUTE, {
      image: imageFile.value
    })

    // Gradio returns result.data as [markdown_string, probabilities_dict]
    const [markdownString, probsDict] = result?.data || ['', {}]
    predictionText.value = typeof markdownString === 'string' ? markdownString : ''

    // Parse a friendlier summary from the markdown result
    try {
      const diseaseMatch = predictionText.value.match(/\*\*Predicted Disease:\*\*\s*(.+)/i)
      const confMatch = predictionText.value.match(/\*\*Confidence:\*\*\s*([0-9.]+)%/i)
      predictedDisease.value = diseaseMatch ? diseaseMatch[1].trim() : ''
      confidencePct.value = confMatch ? Number(confMatch[1]) : 0
      isHealthy.value = /healthy/i.test(predictedDisease.value)
    } catch (_) {
      // Fallbacks already initialized
    }

    const entries = Object.entries(probsDict || {})
      .filter(([label, prob]) => {
        const lname = String(label).toLowerCase()
        const numeric = Number(prob)
        if (['label', 'labels', 'confidence', 'confidences'].includes(lname)) return false
        return Number.isFinite(numeric) && numeric > 0
      })
      .map(([label, prob]) => {
        const numeric = Number(prob)
        return { label, prob: Number.isFinite(numeric) ? numeric : 0 }
      })
      .sort((a, b) => b.prob - a.prob)
      .slice(0, 5)
    top5.value = entries

    // Fallback: if no disease parsed from markdown, use top probability label
    if (!predictedDisease.value && entries.length > 0) {
      predictedDisease.value = String(entries[0].label || '').trim()
      isHealthy.value = /healthy/i.test(predictedDisease.value)
    }

    // Fallback: if no confidence parsed, use top probability * 100
    if ((!confidencePct.value || Number.isNaN(confidencePct.value)) && entries.length > 0) {
      confidencePct.value = (entries[0].prob || 0) * 100
    }

    hasResult.value = true
  } catch (err) {
    errorMessage.value = (err && err.message) ? err.message : 'Failed to analyze image.'
  } finally {
    isLoading.value = false
  }
}

function resetSelection() {
  imageFile.value = null
  predictionText.value = ''
  top5.value = []
  errorMessage.value = ''
  imagePreviewUrl.value = ''
  predictedDisease.value = ''
  confidencePct.value = 0
  isHealthy.value = false
  hasResult.value = false
  if (fileInput.value) {
    fileInput.value.value = ''
  }
}
</script>

<template>
  <div class="container">
    <h1 class="title">🌿 Plant Disease Detection</h1>
    <p class="subtitle">Upload a plant leaf image to detect diseases using our ML model hosted on Hugging Face Spaces.</p>

    <div class="card">
      <div class="uploader">
        <input ref="fileInput" type="file" accept="image/*" @change="onFileChange" style="display:none" />

        <div class="dropzone"
             :class="{ active: isDragActive }"
             @dragover.prevent="onDragOver"
             @dragleave.prevent="onDragLeave"
             @drop.prevent="onDrop"
             @click="openFileDialog">
          <div class="dz-inner">
            <div class="dz-icon">📷</div>
            <div class="dz-text">
              <strong>Drag & drop</strong> an image here, or
              <button type="button" class="linklike" :disabled="isLoading">Choose Image</button>
            </div>
            <div class="dz-help">PNG, JPG up to ~10MB</div>
          </div>
        </div>

        <div class="actions">
          <button class="primary" :disabled="!imageFile || isLoading" @click="analyze">
            <span v-if="!isLoading">🔍 Analyze Image</span>
            <span v-else>Analyzing…</span>
          </button>
          <button class="secondary" :disabled="isLoading && !imageFile" @click="resetSelection">Reset</button>
        </div>
      </div>

      <div class="content">
        <div class="preview" v-if="imagePreviewUrl">
          <img :src="imagePreviewUrl" alt="Preview" />
        </div>

        <div class="results">
          <p v-if="errorMessage" class="error">{{ errorMessage }}</p>

          <div v-if="hasResult" class="summary">
            <div class="summary-header">
              <span class="badge" :class="isHealthy ? 'ok' : 'warn'">{{ isHealthy ? 'Healthy' : 'Disease detected' }}</span>
              <h3 :class="['disease', { danger: !isHealthy }]">{{ (predictedDisease || 'Unknown').replaceAll('_', ' ') }}</h3>
            </div>
            <div class="conf-row">
              <span class="conf-label">Confidence</span>
              <span :class="['conf-value', { danger: !isHealthy }]">{{ confidencePct.toFixed(2) }}%</span>
            </div>
            <div class="conf-bar">
              <span class="conf-fill" :style="{ width: confidencePct + '%' }"></span>
            </div>
          </div>

          <div v-if="top5.length" class="probs">
            <h3>Top 5 Predictions</h3>
            <ul>
              <li v-for="item in top5" :key="item.label">
                <span class="label">{{ (item.label || '').replaceAll('_', ' ') }}</span>
                <span class="bar">
                  <span class="fill" :style="{ width: (item.prob * 100).toFixed(2) + '%' }"></span>
                </span>
                <span class="value">{{ Number.isFinite(item.prob) ? (item.prob * 100).toFixed(2) + '%' : '—' }}</span>
              </li>
            </ul>
          </div>
        </div>
      </div>

      <div v-if="isLoading" class="loading-overlay" aria-live="polite" aria-busy="true">
        <div class="spinner"></div>
        <div>Running analysis…</div>
      </div>
    </div>

    <details class="labels">
      <summary>Supported labels ({{ CLASS_NAMES.length }})</summary>
      <ul>
        <li v-for="name in CLASS_NAMES" :key="name">{{ name.replaceAll('_', ' ') }}</li>
      </ul>
    </details>
    <p class="footnote">Backend: <a href="https://gamika99-plant-disease.hf.space" target="_blank" rel="noreferrer">Hugging Face Space</a> • Endpoint: <code>/predict_image</code></p>
  </div>
  
</template>

<style scoped>
.container {
  max-width: 960px;
  margin: 0 auto;
  padding: 24px;
}
.title {
  margin: 0 0 8px;
}
.subtitle {
  margin: 0 0 16px;
  color: #666;
}
.card {
  border: 1px solid #e5e5e5;
  border-radius: 12px;
  padding: 16px;
  background: #fff;
  box-shadow: 0 2px 8px rgba(0,0,0,0.04);
  position: relative;
}
.uploader {
  display: flex;
  align-items: center;
  gap: 12px;
  flex-wrap: wrap;
}
.dropzone {
  flex: 1 1 100%;
  border: 2px dashed #cbd5e1;
  border-radius: 12px;
  padding: 20px;
  cursor: pointer;
  background: #f8fafc;
  transition: all .15s ease;
}
.dropzone.active {
  border-color: #3b82f6;
  background: #eff6ff;
}
.dz-inner { display:flex; flex-direction:column; align-items:center; gap:6px; }
.dz-icon { font-size: 28px; }
.dz-text { color:#334155; }
.dz-text .linklike { background:none; border:none; color:#2563eb; text-decoration:underline; cursor:pointer; padding:0; }
.dz-help { font-size: 12px; color:#64748b; }
.actions {
  display: flex;
  gap: 8px;
}
button {
  appearance: none;
  border: none;
  background: #3b82f6;
  color: white;
  padding: 8px 14px;
  border-radius: 8px;
  cursor: pointer;
}
button[disabled] {
  opacity: 0.6;
  cursor: not-allowed;
}
button.secondary {
  background: #e5e7eb;
  color: #111827;
}
button.primary {
  background: #2563eb;
}
.content {
  margin-top: 16px;
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
}
.preview {
  border: 1px solid #eee;
  border-radius: 8px;
  padding: 8px;
  background: #fafafa;
}
.preview img {
  max-width: 100%;
  border-radius: 6px;
  display: block;
}
.results {
  display: flex;
  flex-direction: column;
  gap: 12px;
}
.summary {
  border: 1px solid #e5e7eb;
  border-radius: 10px;
  padding: 12px;
  background: #f9fafb;
}
.summary-header { display:flex; align-items:center; gap:8px; margin-bottom:8px; }
.badge { font-size: 12px; padding: 2px 8px; border-radius:999px; border:1px solid transparent; }
.badge.ok { background:#ecfdf5; color:#065f46; border-color:#a7f3d0; }
.badge.warn { background:#fff7ed; color:#9a3412; border-color:#fed7aa; }
.disease { margin:0; font-size: 20px; font-weight: 700; color:#0b1220; }
.disease.danger { color:#b91c1c; }
.conf-row { display:flex; justify-content:space-between; font-size:14px; color:#334155; }
.conf-value { font-weight: 700; color:#0b1220; }
.conf-value.danger { color:#b91c1c; }
.conf-bar { height: 10px; background:#e5e7eb; border-radius:999px; overflow:hidden; margin-top:6px; }
.conf-fill { display:block; height:100%; background:#2563eb; }
.error {
  color: #b91c1c;
}
.prediction {
  white-space: pre-wrap;
  background: #f8fafc;
  padding: 12px;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
}
.probs ul {
  list-style: none;
  padding: 0;
  margin: 0;
  display: flex;
  flex-direction: column;
  gap: 8px;
}
.probs li {
  display: grid;
  grid-template-columns: 2fr 5fr 1fr;
  align-items: center;
  gap: 8px;
}
.label {
  font-size: 14px;
  color:#0b1220;
  font-weight: 600;
}
.bar {
  height: 10px;
  background: #e5e7eb;
  border-radius: 999px;
  overflow: hidden;
}
.fill {
  display: block;
  height: 100%;
  background: #10b981;
}
.value {
  text-align: right;
  font-variant-numeric: tabular-nums;
  color:#0b1220;
  font-weight: 600;
}
.footnote {
  margin-top: 12px;
  color: #666;
}
.labels { margin-top: 16px; }
.labels summary { cursor: pointer; color:#334155; }
.labels ul { margin: 8px 0 0; padding-left: 20px; }
.labels li { margin: 2px 0; font-size: 14px; }
.loading-overlay {
  position:absolute;
  inset: 0;
  background: rgba(255,255,255,0.7);
  display:flex;
  flex-direction:column;
  align-items:center;
  justify-content:center;
  gap:8px;
  border-radius: 12px;
}
.spinner {
  width: 28px;
  height: 28px;
  border: 3px solid #cbd5e1;
  border-top-color: #2563eb;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}
@keyframes spin {
  to { transform: rotate(360deg); }
}
@media (max-width: 800px) {
  .content {
    grid-template-columns: 1fr;
  }
}
</style>


