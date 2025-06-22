<template>
  <section class="hero-container">
    <!-- 3D 粒子背景 canvas 容器 -->
    <div ref="containerRef" class="hero-canvas"></div>
    <!-- 居中内容 -->
    <div class="hero-content">
      <h1 class="hero-title">欢迎来到我的世界</h1>
      <p class="hero-subtitle">前端开发者 | 3D爱好者 | 创意探索者</p>
      <div class="hero-cta">
        <button class="btn primary" @click="scrollToSection('projects')">查看作品</button>
        <button class="btn secondary" @click="scrollToSection('skills')">技能展示</button>
      </div>
    </div>
  </section>
</template>

<script lang="ts" setup>
import { onMounted, ref, onUnmounted } from 'vue'
import * as THREE from 'three'

const containerRef = ref<HTMLElement | null>(null)
let scene: THREE.Scene
let camera: THREE.PerspectiveCamera
let renderer: THREE.WebGLRenderer
let particles: THREE.Points
let animationId: number

const initThree = () => {
  scene = new THREE.Scene()
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
  camera.position.z = 5

  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setClearColor(0x000000, 0) // 完全透明
  containerRef.value?.appendChild(renderer.domElement)

  // 粒子
  const particleCount = 1000
  const positions = new Float32Array(particleCount * 3)
  for (let i = 0; i < particleCount * 3; i++) {
    positions[i] = (Math.random() - 0.5) * 10
  }
  const geometry = new THREE.BufferGeometry()
  geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))
  const material = new THREE.PointsMaterial({ color: 0x667eea, size: 0.05 })
  particles = new THREE.Points(geometry, material)
  scene.add(particles)

  animate()
}

const animate = () => {
  animationId = requestAnimationFrame(animate)
  particles.rotation.y += 0.001
  renderer.render(scene, camera)
}

const handleResize = () => {
  if (!containerRef.value) return
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix()
  renderer.setSize(window.innerWidth, window.innerHeight)
}

const scrollToSection = (id: string) => {
  const el = document.getElementById(id)
  if (el) el.scrollIntoView({ behavior: 'smooth' })
}

onMounted(() => {
  initThree()
  window.addEventListener('resize', handleResize)
})
onUnmounted(() => {
  if (animationId) cancelAnimationFrame(animationId)
  window.removeEventListener('resize', handleResize)
  renderer?.dispose()
})
</script>

<style scoped>
.hero-container {
  position: relative;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #000;
}
.hero-canvas {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
  pointer-events: none;
}
.hero-content {
  position: relative;
  z-index: 1;
  width: 100%;
  text-align: center;
  color: #fff;
}
.hero-title {
  font-size: 3rem;
  margin-bottom: 1rem;
  background: linear-gradient(45deg, #667eea 0%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}
.hero-subtitle {
  font-size: 1.5rem;
  margin-bottom: 2rem;
  color: rgba(255,255,255,0.8);
}
.hero-cta {
  display: flex;
  gap: 1rem;
  justify-content: center;
}
.btn {
  padding: 1rem 2rem;
  border: none;
  border-radius: 50px;
  font-size: 1.1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
}
.btn.primary {
  background: linear-gradient(45deg, #667eea 0%, #764ba2 100%);
  color: white;
}
.btn.secondary {
  background: transparent;
  color: white;
  border: 2px solid rgba(255,255,255,0.3);
}
.btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 10px 30px rgba(0,0,0,0.3);
}
@media (max-width: 768px) {
  .hero-title { font-size: 2rem; }
  .hero-subtitle { font-size: 1rem; }
  .hero-cta { flex-direction: column; }
}
</style>