<template>
  <section id="hero" class="hero-section">
    <!-- Three.js 背景 -->
    <div ref="threeBackground" class="three-background"></div>

    <div class="hero-content">
      <h1 class="hero-title">欢迎来到我的个人主页</h1>
      <p class="hero-subtitle">前端开发工程师 | 3D可视化专家 | 全栈开发者</p>
      <div class="hero-buttons">
        <button class="hero-btn primary" @click="scrollToSection('skills')">
          <span class="btn-icon">💻</span>
          技能展示
        </button>
        <button class="hero-btn secondary" @click="scrollToSection('projects')">
          <span class="btn-icon">🚀</span>
          项目展示
        </button>
        <button class="hero-btn secondary" @click="scrollToSection('experience')">
          <span class="btn-icon">📄</span>
          个人简历
        </button>
      </div>
    </div>
  </section>
</template>

<script lang="ts" setup>
import { onMounted, ref, onUnmounted } from 'vue'
import * as THREE from 'three'

const threeBackground = ref<HTMLElement | null>(null)
let renderer: THREE.WebGLRenderer | null = null
let scene: THREE.Scene
let camera: THREE.PerspectiveCamera
let animationId: number

const scrollToSection = (sectionId: string) => {
  const element = document.getElementById(sectionId)
  if (element) {
    element.scrollIntoView({
      behavior: 'smooth',
      block: 'start'
    })
  }
}

// 初始化 Three.js 背景
const initThreeBackground = () => {
  if (!threeBackground.value) return

  const width = threeBackground.value.clientWidth
  const height = threeBackground.value.clientHeight

  // 创建场景
  scene = new THREE.Scene()

  // 创建相机
  camera = new THREE.PerspectiveCamera(75, width / height, 0.1, 1000)
  camera.position.z = 5

  // 创建渲染器
  renderer = new THREE.WebGLRenderer({
    alpha: true,
    antialias: true
  })
  renderer.setSize(width, height)
  renderer.setClearColor(0x000000, 0)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  threeBackground.value.appendChild(renderer.domElement)

  // 创建粒子系统
  createParticleSystem()

  // 创建几何体
  createGeometries()

  // 开始动画
  animate()
}

// 创建粒子系统
const createParticleSystem = () => {
  const particleCount = 1000
  const positions = new Float32Array(particleCount * 3)
  const colors = new Float32Array(particleCount * 3)

  for (let i = 0; i < particleCount; i++) {
    // 随机位置
    positions[i * 3] = (Math.random() - 0.5) * 20
    positions[i * 3 + 1] = (Math.random() - 0.5) * 20
    positions[i * 3 + 2] = (Math.random() - 0.5) * 20

    // 随机颜色
    const color = new THREE.Color()
    color.setHSL(Math.random() * 0.3 + 0.6, 0.8, 0.5) // 蓝色到紫色范围
    colors[i * 3] = color.r
    colors[i * 3 + 1] = color.g
    colors[i * 3 + 2] = color.b
  }

  const geometry = new THREE.BufferGeometry()
  geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))
  geometry.setAttribute('color', new THREE.BufferAttribute(colors, 3))

  const material = new THREE.PointsMaterial({
    size: 0.05,
    vertexColors: true,
    transparent: true,
    opacity: 0.8,
    blending: THREE.AdditiveBlending
  })

  const particles = new THREE.Points(geometry, material)
  scene.add(particles)
}

// 创建几何体
const createGeometries = () => {
  // 创建多个旋转的几何体
  const geometries = [
    new THREE.TorusGeometry(1, 0.3, 8, 16),
    new THREE.OctahedronGeometry(0.8),
    new THREE.TetrahedronGeometry(0.6)
  ]

  geometries.forEach((geometry, index) => {
    const material = new THREE.MeshBasicMaterial({
      color: new THREE.Color().setHSL(0.6 + index * 0.1, 0.8, 0.5),
      transparent: true,
      opacity: 0.1,
      wireframe: true
    })

    const mesh = new THREE.Mesh(geometry, material)
    mesh.position.set(
      (Math.random() - 0.5) * 8,
      (Math.random() - 0.5) * 8,
      (Math.random() - 0.5) * 8
    )
    mesh.userData = {
      rotationSpeed: {
        x: (Math.random() - 0.5) * 0.02,
        y: (Math.random() - 0.5) * 0.02,
        z: (Math.random() - 0.5) * 0.02
      }
    }
    scene.add(mesh)
  })
}

// 动画循环
const animate = () => {
  animationId = requestAnimationFrame(animate)

  // 旋转场景
  scene.rotation.y += 0.002
  scene.rotation.x += 0.001

  // 动画几何体
  scene.children.forEach((child) => {
    if (child instanceof THREE.Mesh && child.userData.rotationSpeed) {
      const speed = child.userData.rotationSpeed
      child.rotation.x += speed.x
      child.rotation.y += speed.y
      child.rotation.z += speed.z
    }
  })

  // 相机轻微移动
  const time = Date.now() * 0.001
  camera.position.x = Math.sin(time * 0.5) * 0.5
  camera.position.y = Math.cos(time * 0.3) * 0.3

  renderer?.render(scene, camera)
}

// 处理窗口大小变化
const handleResize = () => {
  if (!threeBackground.value || !renderer || !camera) return

  const width = threeBackground.value.clientWidth
  const height = threeBackground.value.clientHeight

  camera.aspect = width / height
  camera.updateProjectionMatrix()
  renderer.setSize(width, height)
}

onMounted(() => {
  setTimeout(() => {
    initThreeBackground()
  }, 100)

  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  if (animationId) {
    cancelAnimationFrame(animationId)
  }
  if (renderer) {
    renderer.dispose()
  }
  window.removeEventListener('resize', handleResize)
})
</script>

<style scoped>
.hero-section {
  width: 100vw;
  height: 100vh;
  background: linear-gradient(135deg, #0f1419 0%, #1a1f2e 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  overflow: hidden;
}

.three-background {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1;
}

.hero-content {
  text-align: center;
  z-index: 2;
  position: relative;
}

.hero-title {
  font-size: 3.5rem;
  font-weight: 700;
  color: #fff;
  margin-bottom: 1rem;
  text-shadow: 0 0 30px rgba(102, 126, 234, 0.5);
  animation: fadeInUp 1s ease-out;
}

.hero-subtitle {
  font-size: 1.3rem;
  color: #ccc;
  margin-bottom: 3rem;
  animation: fadeInUp 1s ease-out 0.2s both;
}

.hero-buttons {
  display: flex;
  gap: 1.5rem;
  justify-content: center;
  animation: fadeInUp 1s ease-out 0.4s both;
  flex-wrap: wrap;
}

.hero-btn {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 1rem 2rem;
  border: none;
  border-radius: 30px;
  font-size: 1.1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  text-decoration: none;
  position: relative;
  overflow: hidden;
  backdrop-filter: blur(10px);
}

.hero-btn::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255,255,255,0.2), transparent);
  transition: left 0.5s ease;
}

.hero-btn:hover::before {
  left: 100%;
}

.hero-btn.primary {
  background: linear-gradient(135deg, #667eea, #42b883);
  color: #fff;
  box-shadow: 0 4px 20px rgba(102, 126, 234, 0.4);
}

.hero-btn.primary:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 30px rgba(102, 126, 234, 0.6);
}

.hero-btn.secondary {
  background: rgba(255, 255, 255, 0.1);
  color: #fff;
  border: 2px solid rgba(255, 255, 255, 0.2);
}

.hero-btn.secondary:hover {
  background: rgba(255, 255, 255, 0.2);
  border-color: rgba(255, 255, 255, 0.4);
  transform: translateY(-3px);
  box-shadow: 0 8px 30px rgba(255, 255, 255, 0.2);
}

.btn-icon {
  font-size: 1.2rem;
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@media (max-width: 768px) {
  .hero-title {
    font-size: 2.5rem;
  }

  .hero-subtitle {
    font-size: 1.1rem;
  }

  .hero-buttons {
    flex-direction: column;
    align-items: center;
    gap: 1rem;
  }

  .hero-btn {
    width: 250px;
    justify-content: center;
  }
}

@media (max-width: 480px) {
  .hero-title {
    font-size: 2rem;
  }

  .hero-subtitle {
    font-size: 1rem;
  }

  .hero-btn {
    width: 200px;
    padding: 0.8rem 1.5rem;
    font-size: 1rem;
  }
}
</style>