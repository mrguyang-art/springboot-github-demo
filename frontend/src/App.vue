<script setup>
import { computed, onMounted, ref } from 'vue'
import axios from 'axios'
import {
  Connection, DataBoard, Document, Refresh, CircleCheck,
  Monitor, Promotion, Cpu, Link
} from '@element-plus/icons-vue'

const apiBase = import.meta.env.VITE_API_BASE_URL || ''
const apiResult = ref(null)
const health = ref(null)
const loading = ref(false)
const errorMessage = ref('')
const lastUpdated = ref('尚未请求')

const apiUrl = computed(() => apiBase ? `${apiBase}/api/hello` : '/api/hello')

async function loadDemo() {
  loading.value = true
  errorMessage.value = ''
  try {
    const [helloResponse, healthResponse] = await Promise.all([
      axios.get(`${apiBase}/api/hello`),
      axios.get(`${apiBase}/actuator/health`).catch(() => null)
    ])
    apiResult.value = helloResponse.data
    health.value = healthResponse?.data?.status || 'UNKNOWN'
    lastUpdated.value = new Date().toLocaleString('zh-CN')
  } catch (error) {
    apiResult.value = null
    health.value = 'UNAVAILABLE'
    errorMessage.value = error.response
      ? `后端返回 HTTP ${error.response.status}`
      : '无法连接 Spring Boot。请确认后端已在 localhost:8080 启动。'
  } finally {
    loading.value = false
  }
}

function openDocs() {
  window.open('https://mrguyang-art.github.io/springboot-github-demo/', '_blank', 'noopener')
}

onMounted(loadDemo)
</script>

<template>
  <div class="app-shell">
    <header class="topbar">
      <div class="brand">
        <div class="brand-mark"><Connection /></div>
        <div>
          <div class="brand-name">SPRING<span>·</span>VUE</div>
          <div class="brand-caption">FULL-STACK DEMO</div>
        </div>
      </div>
      <div class="topbar-right">
        <el-tag type="success" effect="light" round><span class="live-dot"></span>前后端分离</el-tag>
        <span class="topbar-version">DEMO 1.1</span>
      </div>
    </header>

    <main class="main-content">
      <section class="hero">
        <div class="hero-copy">
          <div class="eyebrow"><span></span> DECOUPLED APPLICATION ARCHITECTURE</div>
          <h1>前端专注体验，<br /><em>后端专注服务。</em></h1>
          <p>Vue 3 + Element Plus 构建独立前端，通过 REST API 与 Spring Boot 通信。两个应用独立开发、独立构建、独立部署。</p>
          <div class="hero-actions">
            <el-button type="primary" size="large" :loading="loading" @click="loadDemo">
              <el-icon><Refresh /></el-icon> 请求后端 API
            </el-button>
            <el-button size="large" plain @click="openDocs">
              静态项目文档 <el-icon><Promotion /></el-icon>
            </el-button>
          </div>
        </div>
        <div class="hero-visual" aria-label="前后端分离示意">
          <div class="orbit orbit-one"></div><div class="orbit orbit-two"></div>
          <div class="node node-vue"><span>V</span><small>VUE 3</small></div>
          <div class="node node-api"><el-icon><Link /></el-icon><small>REST API</small></div>
          <div class="node node-java"><span>☕</span><small>SPRING BOOT</small></div>
          <div class="connection-line"></div>
          <div class="visual-label">HTTP / JSON</div>
        </div>
      </section>

      <section class="metrics">
        <article class="metric-card">
          <div class="metric-icon purple"><el-icon><Monitor /></el-icon></div>
          <div><div class="metric-label">前端应用</div><div class="metric-value">Vue 3</div><div class="metric-sub">Element Plus · Vite</div></div>
        </article>
        <article class="metric-card">
          <div class="metric-icon green"><el-icon><Cpu /></el-icon></div>
          <div><div class="metric-label">后端服务</div><div class="metric-value">Spring Boot</div><div class="metric-sub">Java 21 · REST API</div></div>
        </article>
        <article class="metric-card">
          <div class="metric-icon blue"><el-icon><DataBoard /></el-icon></div>
          <div><div class="metric-label">通信协议</div><div class="metric-value">HTTP + JSON</div><div class="metric-sub">跨域配置 / Vite Proxy</div></div>
        </article>
        <article class="metric-card">
          <div class="metric-icon orange"><el-icon><Document /></el-icon></div>
          <div><div class="metric-label">静态文档</div><div class="metric-value">GitHub Pages</div><div class="metric-sub">docs/ 独立托管</div></div>
        </article>
      </section>

      <section class="content-grid">
        <el-card class="api-card" shadow="never">
          <template #header>
            <div class="card-heading"><div><div class="section-kicker">LIVE API RESPONSE</div><h2>后端接口联调</h2></div><el-button text type="primary" :loading="loading" @click="loadDemo"><el-icon><Refresh /></el-icon>刷新</el-button></div>
          </template>
          <div class="endpoint"><span class="method">GET</span><code>{{ apiUrl }}</code><el-tag size="small" type="success">REST</el-tag></div>
          <div v-if="apiResult" class="response success-response">
            <div class="response-title"><el-icon><CircleCheck /></el-icon> 请求成功 · 200 OK</div>
            <pre>{{ JSON.stringify(apiResult, null, 2) }}</pre>
          </div>
          <el-alert v-else-if="errorMessage" :title="errorMessage" type="warning" :closable="false" show-icon />
          <el-skeleton v-else :rows="3" animated />
          <div class="api-footer"><span>最近请求：{{ lastUpdated }}</span><span>GET /actuator/health：<b :class="health === 'UP' ? 'healthy' : 'unhealthy'">{{ health || '检查中' }}</b></span></div>
        </el-card>

        <el-card class="architecture-card" shadow="never">
          <template #header><div class="section-kicker">PROJECT STRUCTURE</div><h2>工程边界清晰</h2></template>
          <div class="layer"><div class="layer-icon frontend-icon">V</div><div class="layer-info"><b>frontend/</b><span>Vue 页面、组件、交互逻辑</span></div><el-tag type="primary">独立构建</el-tag></div>
          <div class="layer-arrow">↓ HTTP / JSON ↓</div>
          <div class="layer"><div class="layer-icon backend-icon">☕</div><div class="layer-info"><b>src/main/</b><span>Spring Boot Controller、业务接口</span></div><el-tag type="success">独立运行</el-tag></div>
          <div class="layer-divider"></div>
          <div class="docs-note"><el-icon><Document /></el-icon><span><b>docs/</b> 保持纯静态 HTML/CSS，继续由 GitHub Pages 发布，不参与 Vue 构建。</span></div>
        </el-card>
      </section>

      <footer class="page-footer"><span>SPRINGBOOT GITHUB DEMO</span><span>前端与后端解耦 · 文档站点独立</span></footer>
    </main>
  </div>
</template>
