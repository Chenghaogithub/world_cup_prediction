<template>
  <div class="app">
    <!-- 足球场背景 -->
    <div class="field-bg">
      <div class="field-lines">
        <div class="center-circle"></div>
        <div class="center-line"></div>
      </div>
    </div>

    <!-- 浮动足球装饰 -->
    <div class="floating-balls">
      <span v-for="n in 6" :key="n" class="ball" :style="ballStyle(n)">⚽</span>
    </div>

    <!-- 烟花画布 -->
    <canvas ref="fireworksCanvas" class="fireworks-canvas"></canvas>

    <!-- 左上角AI预测标签 -->
    <div class="top-left-tag">🤖 AI预测</div>

    <div class="content">
      <!-- 标题区域 -->
      <div class="header">
        <div class="trophy">🏆</div>
        <h1 class="title">
          <span class="title-year">2026</span>
          <span class="title-main">美加墨世界杯</span>
          <span class="title-sub">夺冠概率预测</span>
          
        </h1>
        <div class="ai-badge">
          <span class="ai-dot"></span>
          AI 大模型预测
        </div>
      </div>

      <!-- 预测按钮 -->
      <div v-if="!isPredicting && !showResult" class="predict-section">
        <button class="predict-btn" @click="startPrediction">
          <span class="btn-icon">🤖</span>
          <span class="btn-text">一键AI预测</span>
          <span class="btn-glow"></span>
        </button>
        <p class="hint">基于深度学习的超精准预测</p>
      </div>

      <!-- 进度区域 -->
      <div v-if="isPredicting" class="progress-section">
        <div class="progress-container">
          <div class="progress-bar">
            <div class="progress-fill" :style="{ width: progress + '%' }">
              <div class="progress-shine"></div>
            </div>
          </div>
          <div class="progress-text">{{ Math.floor(progress) }}%</div>
        </div>

        <div class="stage-list">
          <transition name="stage" mode="out-in">
            <div
              v-if="completing"
              key="completing"
              class="stage-item active"
            >
            
              <span class="stage-text">模拟完成，请稍后...</span>
            </div>
            <div
              v-else-if="currentStageText"
              :key="currentStage"
              class="stage-item active"
            >
              <span class="stage-icon">
                <span class="loading-spinner"></span>
              </span>
              <span class="stage-text">{{ currentStageText }}</span>
            </div>
          </transition>
        </div>
      </div>

      <!-- 结果区域 -->
      <div v-if="showResult" class="result-section">
        <div class="result-card" :class="{ 'show': showResult }">
          <div class="result-header">
            <span class="sparkle">✨</span>
            恭喜，预测完成
            <span class="sparkle">✨</span>
          </div>
          <div class="result-body">
            <div class="result-info">
              <div class="result-row">
                <span class="result-label">夺冠球队：</span>
                <span class="result-value">国足</span>
              </div>
              <div class="result-row">
                <span class="result-label">概率：</span>
                <span class="result-value probability-zero">0%</span>
              </div>
            </div>
          </div>
          <div class="result-footer">
            ⚠️ 本预测由AI生成，准确率高达100% ⚠️
          </div>
        </div>

      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, watch, onUnmounted, nextTick } from 'vue'

export default {
  name: 'App',
  setup() {
    const isPredicting = ref(false)
    const completing = ref(false)
    const showResult = ref(false)
    const progress = ref(0)
    const currentStage = ref(-1)
    const fireworksCanvas = ref(null)

    // --- 烟花系统 ---
    let ctx = null
    let animFrameId = null
    let fireworks = []
    let particles = []

    class Firework {
      constructor(w, h) {
        this.x = w * (0.2 + Math.random() * 0.6)
        this.y = h
        this.targetY = h * (0.1 + Math.random() * 0.35)
        this.speed = 4 + Math.random() * 3
        this.hue = Math.floor(Math.random() * 360)
        this.alive = true
        this.trail = []
      }
      update() {
        this.trail.push({ x: this.x, y: this.y })
        if (this.trail.length > 6) this.trail.shift()
        this.y -= this.speed
        if (this.y <= this.targetY) {
          this.alive = false
          this.explode()
        }
      }
      draw(c) {
        for (let i = 0; i < this.trail.length; i++) {
          const a = i / this.trail.length
          c.beginPath()
          c.arc(this.trail[i].x, this.trail[i].y, 2, 0, Math.PI * 2)
          c.fillStyle = `hsla(${this.hue}, 100%, 70%, ${a * 0.5})`
          c.fill()
        }
        c.beginPath()
        c.arc(this.x, this.y, 3, 0, Math.PI * 2)
        c.fillStyle = `hsl(${this.hue}, 100%, 80%)`
        c.fill()
      }
      explode() {
        const count = 60 + Math.floor(Math.random() * 40)
        for (let i = 0; i < count; i++) {
          const angle = (Math.PI * 2 * i) / count + (Math.random() - 0.5) * 0.5
          const speed = 2 + Math.random() * 4
          particles.push(new Particle(this.x, this.y, angle, speed, this.hue))
        }
      }
    }

    class Particle {
      constructor(x, y, angle, speed, hue) {
        this.x = x
        this.y = y
        this.vx = Math.cos(angle) * speed
        this.vy = Math.sin(angle) * speed
        this.hue = hue + Math.floor(Math.random() * 30 - 15)
        this.alpha = 1
        this.decay = 0.015 + Math.random() * 0.02
        this.size = 2 + Math.random() * 2
        this.gravity = 0.04
      }
      update() {
        this.vx *= 0.98
        this.vy *= 0.98
        this.vy += this.gravity
        this.x += this.vx
        this.y += this.vy
        this.alpha -= this.decay
      }
      draw(c) {
        c.beginPath()
        c.arc(this.x, this.y, this.size, 0, Math.PI * 2)
        c.fillStyle = `hsla(${this.hue}, 100%, 65%, ${this.alpha})`
        c.fill()
      }
    }

    let launchTimer = null

    function startFireworks() {
      const canvas = fireworksCanvas.value
      if (!canvas) return
      canvas.width = window.innerWidth
      canvas.height = window.innerHeight
      ctx = canvas.getContext('2d')
      fireworks = []
      particles = []

      let launchCount = 0
      const maxLaunches = 15

      function launchOne() {
        if (launchCount < maxLaunches) {
          fireworks.push(new Firework(canvas.width, canvas.height))
          launchCount++
          launchTimer = setTimeout(launchOne, 300 + Math.random() * 500)
        }
      }
      launchOne()

      function animate() {
        ctx.globalCompositeOperation = 'source-over'
        ctx.fillStyle = 'rgba(10, 22, 40, 0.4)'
        ctx.fillRect(0, 0, canvas.width, canvas.height)
        ctx.globalCompositeOperation = 'lighter'

        for (let i = fireworks.length - 1; i >= 0; i--) {
          fireworks[i].update()
          fireworks[i].draw(ctx)
          if (!fireworks[i].alive) fireworks.splice(i, 1)
        }

        for (let i = particles.length - 1; i >= 0; i--) {
          particles[i].update()
          particles[i].draw(ctx)
          if (particles[i].alpha <= 0) particles.splice(i, 1)
        }

        animFrameId = requestAnimationFrame(animate)
      }
      animate()
    }

    function stopFireworks() {
      if (animFrameId) cancelAnimationFrame(animFrameId)
      if (launchTimer) clearTimeout(launchTimer)
      animFrameId = null
      launchTimer = null
      fireworks = []
      particles = []
      const canvas = fireworksCanvas.value
      if (canvas) {
        const c = canvas.getContext('2d')
        c.clearRect(0, 0, canvas.width, canvas.height)
      }
    }

    watch(showResult, (val) => {
      if (val) {
        nextTick(() => startFireworks())
      } else {
        stopFireworks()
      }
    })

    onUnmounted(() => stopFireworks())

    const stages = [
      '正在录入并分析48支球队信息 ...',
      '正在联网搜索球员最新伤停状态 ...',
      '正在推演1400万种比赛走向 ...'
    ]

    const currentStageText = computed(() => {
      if (currentStage.value < 0 || currentStage.value >= stages.length) return ''
      return stages[currentStage.value]
    })

    function startPrediction() {
      isPredicting.value = true
      showResult.value = false
      progress.value = 0
      currentStage.value = 0

      const totalDuration = 10000
      const interval = 50
      const increment = 100 / (totalDuration / interval)
      const stageThresholds = [25, 50]

      const timer = setInterval(() => {
        progress.value += increment + Math.random() * 0.3

        if (progress.value > 100) {
          progress.value = 100
        }

        for (let i = 0; i < stageThresholds.length; i++) {
          if (progress.value >= stageThresholds[i] && currentStage.value <= i) {
            currentStage.value = i + 1
            break
          }
        }

        if (progress.value >= 100) {
          currentStage.value = -1
          completing.value = true
          clearInterval(timer)
          setTimeout(() => {
            completing.value = false
            isPredicting.value = false
            showResult.value = true
          }, 1500)
        }
      }, interval)
    }

    function ballStyle(n) {
      const positions = [
        { left: '5%', top: '10%', delay: '0s', size: '48px' },
        { left: '85%', top: '15%', delay: '1.5s', size: '40px' },
        { left: '10%', top: '70%', delay: '3s', size: '34px' },
        { left: '90%', top: '65%', delay: '0.8s', size: '52px' },
        { left: '50%', top: '5%', delay: '2s', size: '36px' },
        { left: '45%', top: '85%', delay: '4s', size: '44px' },
      ]
      const p = positions[n - 1]
      return {
        left: p.left,
        top: p.top,
        animationDelay: p.delay,
        fontSize: p.size,
      }
    }

    return {
      isPredicting,
      completing,
      showResult,
      progress,
      currentStage,
      stages,
      currentStageText,
      startPrediction,
      ballStyle,
      fireworksCanvas,
    }
  }
}
</script>

<style>
/* Reset & Base */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Microsoft YaHei', 'PingFang SC', sans-serif;
  overflow: hidden;
  background: #0a1628;
}

/* Fireworks Canvas */
.fireworks-canvas {
  position: fixed;
  inset: 0;
  z-index: 1;
  pointer-events: none;
}

/* Main App Container */
.app {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  overflow: hidden;
  background: linear-gradient(135deg, #0a1628 0%, #1a3a2a 50%, #0d2818 100%);
}

/* Football Field Background */
.field-bg {
  position: absolute;
  inset: 0;
  background: linear-gradient(180deg,
    rgba(34, 120, 60, 0.15) 0%,
    rgba(34, 120, 60, 0.08) 50%,
    rgba(34, 120, 60, 0.15) 100%
  );
  pointer-events: none;
}

.field-lines {
  position: absolute;
  inset: 10%;
  border: 2px solid rgba(255, 255, 255, 0.08);
  border-radius: 4px;
}

.center-circle {
  position: absolute;
  width: 120px;
  height: 120px;
  border: 2px solid rgba(255, 255, 255, 0.08);
  border-radius: 50%;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.center-line {
  position: absolute;
  width: 2px;
  height: 100%;
  background: rgba(255, 255, 255, 0.08);
  left: 50%;
  top: 0;
}

/* Floating Footballs */
.floating-balls .ball {
  position: absolute;
  animation: float 6s ease-in-out infinite;
  opacity: 0.3;
  filter: grayscale(30%);
}

@keyframes float {
  0%, 100% { transform: translateY(0) rotate(0deg); }
  50% { transform: translateY(-20px) rotate(180deg); }
}

/* Content */
.content {
  position: relative;
  z-index: 10;
  text-align: center;
  padding: 40px;
  max-width: 700px;
  width: 100%;
}

/* Header */
.header {
  margin-bottom: 50px;
}

.trophy {
  font-size: 72px;
  animation: trophy-bounce 2s ease-in-out infinite;
  filter: drop-shadow(0 0 30px rgba(255, 215, 0, 0.5));
}

@keyframes trophy-bounce {
  0%, 100% { transform: translateY(0) scale(1); }
  50% { transform: translateY(-15px) scale(1.05); }
}

.title {
  color: white;
  margin: 16px 0;
  line-height: 1.3;
}

.title-year {
  display: block;
  font-size: 52px;
  font-weight: 900;
  background: linear-gradient(135deg, #ffd700, #ff8c00, #ffd700);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  text-shadow: none;
  letter-spacing: 4px;
}

.title-main {
  display: block;
  font-size: 38px;
  font-weight: 800;
  color: #fff;
  letter-spacing: 6px;
  text-shadow: 0 2px 20px rgba(255, 255, 255, 0.3);
}

.title-sub {
  display: block;
  font-size: 30px;
  font-weight: 700;
  background: linear-gradient(90deg, #00d4ff, #7b2ff7, #ff2d95);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  letter-spacing: 8px;
  margin-top: 4px;
}

/* Top-left AI Tag */
.top-left-tag {
  position: fixed;
  top: 20px;
  left: 24px;
  z-index: 100;
  padding: 8px 20px;
  font-size: 18px;
  font-weight: 800;
  letter-spacing: 4px;
  color: #fff;
  background: linear-gradient(135deg, #7b2ff7, #ff2d95);
  border-radius: 8px;
  box-shadow: 0 0 20px rgba(123, 47, 247, 0.5), 0 0 40px rgba(255, 45, 149, 0.3);
  animation: ai-tag-glow 2s ease-in-out infinite;
}

@keyframes ai-tag-glow {
  0%, 100% { box-shadow: 0 0 20px rgba(123, 47, 247, 0.5), 0 0 40px rgba(255, 45, 149, 0.3); }
  50% { box-shadow: 0 0 30px rgba(123, 47, 247, 0.7), 0 0 60px rgba(255, 45, 149, 0.5); }
}

.ai-badge {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  margin-top: 16px;
  padding: 6px 20px;
  background: rgba(123, 47, 247, 0.2);
  border: 1px solid rgba(123, 47, 247, 0.4);
  border-radius: 20px;
  color: #b388ff;
  font-size: 13px;
  font-weight: 600;
  letter-spacing: 3px;
}

.ai-dot {
  width: 8px;
  height: 8px;
  background: #7b2ff7;
  border-radius: 50%;
  animation: pulse-dot 1.5s ease-in-out infinite;
}

@keyframes pulse-dot {
  0%, 100% { opacity: 1; transform: scale(1); }
  50% { opacity: 0.4; transform: scale(0.8); }
}

/* Predict Button */
.predict-section {
  margin-top: 20px;
}

.predict-btn {
  position: relative;
  display: inline-flex;
  align-items: center;
  gap: 12px;
  padding: 20px 56px;
  font-size: 24px;
  font-weight: 800;
  color: white;
  background: linear-gradient(135deg, #ff6b35, #f7c948, #ff6b35);
  background-size: 200% 200%;
  border: none;
  border-radius: 60px;
  cursor: pointer;
  transition: all 0.3s ease;
  animation: gradient-shift 3s ease infinite;
  box-shadow:
    0 4px 30px rgba(255, 107, 53, 0.4),
    0 0 60px rgba(247, 201, 72, 0.2);
  overflow: hidden;
}

@keyframes gradient-shift {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

.predict-btn:hover {
  transform: translateY(-3px) scale(1.05);
  box-shadow:
    0 8px 40px rgba(255, 107, 53, 0.6),
    0 0 80px rgba(247, 201, 72, 0.3);
}

.predict-btn:active {
  transform: translateY(0) scale(0.98);
}

.btn-icon {
  font-size: 28px;
}

.btn-glow {
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: radial-gradient(circle, rgba(255,255,255,0.2) 0%, transparent 70%);
  animation: glow-rotate 4s linear infinite;
}

@keyframes glow-rotate {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.hint {
  color: rgba(255, 255, 255, 0.4);
  margin-top: 20px;
  font-size: 14px;
  font-style: italic;
}

/* Progress Section */
.progress-section {
  margin-top: 20px;
  animation: fadeIn 0.5s ease;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}

.progress-container {
  display: flex;
  align-items: center;
  gap: 16px;
  margin-bottom: 36px;
}

.progress-bar {
  flex: 1;
  height: 20px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 10px;
  overflow: hidden;
  position: relative;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #00d4ff, #7b2ff7, #ff2d95);
  background-size: 200% 100%;
  border-radius: 10px;
  transition: width 0.1s linear;
  position: relative;
  animation: progress-gradient 2s linear infinite;
}

@keyframes progress-gradient {
  0% { background-position: 0% 50%; }
  100% { background-position: 200% 50%; }
}

.progress-shine {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(90deg,
    transparent 0%,
    rgba(255, 255, 255, 0.3) 50%,
    transparent 100%
  );
  animation: shine 1.5s ease-in-out infinite;
}

@keyframes shine {
  0% { transform: translateX(-100%); }
  100% { transform: translateX(100%); }
}

.progress-text {
  font-size: 28px;
  font-weight: 900;
  color: #00d4ff;
  min-width: 60px;
  text-shadow: 0 0 20px rgba(0, 212, 255, 0.5);
}

/* Stage List */
.stage-list {
  text-align: left;
  max-width: 480px;
  margin: 0 auto;
}

.stage-item {
  display: flex;
  align-items: center;
  gap: 14px;
  padding: 14px 20px;
  margin-bottom: 8px;
  border-radius: 12px;
  font-size: 16px;
  transition: all 0.4s ease;
  background: rgba(255, 255, 255, 0.03);
}

.stage-item.active {
  background: rgba(123, 47, 247, 0.15);
  border-left: 3px solid #7b2ff7;
  color: #e0d0ff;
  font-weight: 600;
}

.stage-item.done {
  color: rgba(255, 255, 255, 0.5);
}

.stage-item.pending {
  color: rgba(255, 255, 255, 0.2);
}

/* Stage transition */
.stage-enter-active {
  transition: all 0.5s cubic-bezier(0.34, 1.56, 0.64, 1);
}

.stage-enter-from {
  opacity: 0;
  transform: translateX(-30px);
}

.stage-icon {
  flex-shrink: 0;
  width: 24px;
  text-align: center;
}

.loading-spinner {
  display: inline-block;
  width: 18px;
  height: 18px;
  border: 3px solid rgba(123, 47, 247, 0.3);
  border-top-color: #7b2ff7;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

/* Result Section */
.result-section {
  animation: resultAppear 0.8s cubic-bezier(0.34, 1.56, 0.64, 1);
}

@keyframes resultAppear {
  from {
    opacity: 0;
    transform: scale(0.5) translateY(30px);
  }
  to {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}

.result-card {
  background: linear-gradient(145deg,
    rgba(255, 255, 255, 0.1) 0%,
    rgba(255, 255, 255, 0.05) 100%
  );
  backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.15);
  border-radius: 24px;
  padding: 40px;
  box-shadow:
    0 20px 60px rgba(0, 0, 0, 0.3),
    inset 0 1px 0 rgba(255, 255, 255, 0.1);
}

.result-header {
  font-size: 22px;
  font-weight: 700;
  color: #ffd700;
  letter-spacing: 4px;
  margin-bottom: 30px;
}

.sparkle {
  animation: sparkle 1.5s ease-in-out infinite;
}

@keyframes sparkle {
  0%, 100% { opacity: 1; transform: scale(1); }
  50% { opacity: 0.5; transform: scale(1.3); }
}

.result-body {
  margin-bottom: 24px;
}

.result-info {
  display: flex;
  flex-direction: column;
  gap: 16px;
  align-items: center;
}

.result-row {
  display: flex;
  align-items: baseline;
  gap: 4px;
}

.result-label {
  font-size: 24px;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.6);
}

.result-value {
  font-size: 32px;
  font-weight: 800;
  color: white;
  letter-spacing: 4px;
}

.probability-zero {
  font-size: 36px;
  font-weight: 900;
  background: linear-gradient(135deg, #ff4444, #ff6b6b);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.result-footer {
  color: #ffaa00;
  font-size: 16px;
  font-weight: 700;
  border-top: 1px solid rgba(255, 170, 0, 0.2);
  padding-top: 20px;
  margin-top: 20px;
  text-shadow: 0 0 10px rgba(255, 170, 0, 0.3);
}
</style>
