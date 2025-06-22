<template>
  <section id="skills" class="skills-section">
    <!-- 背景 Canvas -->
    <div ref="textParticleBox" class="background-canvas"></div>

    <!-- 内容层 -->
    <div class="content-layer">
      <h2 class="section-title">技能展示</h2>

      <!-- 技能分类切换 -->
      <div class="skill-categories" ref="categoriesContainer">
        <button
          v-for="category in skillCategories"
          :key="category.key"
          class="category-btn"
          :class="{ active: activeCategory === category.key }"
          @click="switchCategory(category.key)"
        >
          <span class="category-icon">{{ category.icon }}</span>
          {{ category.name }}
        </button>
      </div>
    </div>
  </section>
</template>

<script setup lang="ts">
import { onMounted, ref, onUnmounted, computed, nextTick } from 'vue'
import * as THREE from 'three'

const textParticleBox = ref<HTMLElement | null>(null)
const categoriesContainer = ref<HTMLElement | null>(null)
let textParticleRenderer: THREE.WebGLRenderer | null = null
let textParticleScene: THREE.Scene
let textParticleCamera: THREE.PerspectiveCamera
let animationId1: number

// 技能分类
const skillCategories = [
  { key: 'frontend', name: '前端', icon: '💻' },
  { key: '3d', name: '3D & 可视化', icon: '🌐' },
  { key: 'language', name: '开发语言', icon: '📝' },
  { key: 'backend', name: '后端 & 数据库', icon: '🗄️' },
  { key: 'tools', name: '工具 & 部署', icon: '🛠️' }
]

// 各分类的技能关键词和对应的官方颜色
const skillKeywordsByCategory = {
  frontend: ['REACT', 'VUE.JS', 'HTML5', 'CSS3', 'SASS', 'WEBPACK', 'VITE'],
  '3d': ['THREE.JS', 'CESIUMJS', 'D3.JS', 'CANVAS', 'WEBGL', 'WEBGPU'],
  language: ['JAVASCRIPT', 'TYPESCRIPT', 'C++', 'PYTHON'],
  backend: ['NODE.JS', 'EXPRESS', 'MONGODB', 'POSTGRESQL', 'REDIS', 'REST API'],
  tools: ['GIT', 'AWS', 'DOCKER', 'LINUX', 'NPM', 'YARN']
}

// 各技术的官方颜色
const technologyColors = {
  // 前端技术
  'REACT': 0x61dafb,        // React 蓝色
  'VUE.JS': 0x42b883,       // Vue 绿色
  'HTML5': 0xe34c26,        // HTML5 橙色
  'CSS3': 0x1572b6,         // CSS3 蓝色
  'SASS': 0xcc6699,         // Sass 粉色
  'WEBPACK': 0x8dd6f9,      // Webpack 蓝色
  'VITE': 0x646cff,         // Vite 紫色

  // 3D & 可视化
  'THREE.JS': 0x667eea,     // Three.js 蓝色
  'WEBGL': 0x990000,        // WebGL 红色
  'CESIUMJS': 0x6cbd45,     // CesiumJS 绿色
  'D3.JS': 0xff7f0e,        // D3.js 橙色
  'CANVAS': 0xffffff,       // Canvas 白色
  'WEBGPU': 0x00aaff,       // WebGPU 蓝色

  // 开发语言
  'JAVASCRIPT': 0xf7df1e,   // JavaScript 黄色
  'TYPESCRIPT': 0x3178c6,   // TypeScript 蓝色
  'C++': 0x00599c,          // C++ 蓝色
  'PYTHON': 0x3776ab,       // Python 蓝色
  'JAVA': 0xed8b00,         // Java 橙色

  // 后端 & 数据库
  'NODE.JS': 0x3c873a,      // Node.js 绿色
  'EXPRESS': 0x000000,      // Express 黑色
  'MONGODB': 0x4db33d,      // MongoDB 绿色
  'POSTGRESQL': 0x336791,   // PostgreSQL 蓝色
  'REDIS': 0xdc382d,        // Redis 红色
  'REST API': 0x00d4aa,     // REST API 青色

  // 工具 & 部署
  'GIT': 0xf05032,          // Git 橙色
  'AWS': 0xff9900,          // AWS 橙色
  'DOCKER': 0x2496ed,       // Docker 蓝色
  'LINUX': 0xfcc624,        // Linux 黄色
  'NPM': 0xcb3837,          // npm 红色
  'YARN': 0x2c8ebb          // Yarn 蓝色
}

const activeCategory = ref('frontend')

const switchCategory = (categoryKey: string) => {
  activeCategory.value = categoryKey
  if (textParticleScene) {
    textParticleScene.clear()
  }
  initTextParticle()
}

const getActiveCategoryName = () => {
  const category = skillCategories.find(cat => cat.key === activeCategory.value)
  return category ? category.name : ''
}

// 文字粒子化 - 竖着展示
const createTextParticles = (text: string, color: number, offsetX: number, offsetY: number) => {
  const canvas = document.createElement('canvas')
  const ctx = canvas.getContext('2d')
  if (!ctx) return []

  const textLength = text.length
  let canvasWidth = 280
  let canvasHeight = 80
  let fontSize = 50

  if (textLength > 8) {
    canvasWidth = 350
    fontSize = 45
  }

  canvas.width = canvasWidth
  canvas.height = canvasHeight
  ctx.fillStyle = '#fff'
  ctx.font = `bold ${fontSize}px Arial`
  ctx.textAlign = 'center'
  ctx.textBaseline = 'middle'
  ctx.fillText(text, canvas.width / 2, canvas.height / 2)

  const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height)
  const data = imageData.data
  const particles: THREE.Mesh[] = []
  const step = 2

  for (let y = 0; y < canvas.height; y += step) {
    for (let x = 0; x < canvas.width; x += step) {
      const index = (y * canvas.width + x) * 4
      const alpha = data[index + 3]
      if (alpha > 128) {
        const geometry = new THREE.SphereGeometry(0.03, 4, 4)
        const material = new THREE.MeshBasicMaterial({ color, transparent: true, opacity: 0.8 })
        const mesh = new THREE.Mesh(geometry, material)

        mesh.position.set(
          (x - canvas.width / 2) * 0.015 + offsetX,
          (canvas.height / 2 - y) * 0.015 + offsetY,
          (Math.random() - 0.5) * 0.1
        )

        mesh.userData = {
          originalPosition: mesh.position.clone()
        }

        particles.push(mesh)
      }
    }
  }
  return particles
}

const initTextParticle = () => {
  // 获取整个 section 的尺寸
  const sectionElement = document.querySelector('.skills-section') as HTMLElement
  const width = sectionElement ? sectionElement.offsetWidth : window.innerWidth
  const height = sectionElement ? sectionElement.offsetHeight : window.innerHeight

  if (!textParticleScene) {
    textParticleScene = new THREE.Scene()
    textParticleCamera = new THREE.PerspectiveCamera(60, width / height, 0.1, 1000)
    textParticleCamera.position.z = 12
    textParticleRenderer = new THREE.WebGLRenderer({ alpha: true })
    textParticleRenderer.setSize(width, height)
    textParticleRenderer.setClearColor(0x000000, 0)

    if (textParticleBox.value) {
      textParticleBox.value.appendChild(textParticleRenderer.domElement)
    }
  } else {
    textParticleScene.clear()
    textParticleRenderer?.setSize(width, height)
    textParticleCamera.aspect = width / height
    textParticleCamera.updateProjectionMatrix()
  }

  const currentKeywords = skillKeywordsByCategory[activeCategory.value as keyof typeof skillKeywordsByCategory]

  // 竖着展示 - 使用官方颜色
  currentKeywords.forEach((keyword, idx) => {
    // 获取技术的官方颜色，如果没有则使用默认颜色
    const color = technologyColors[keyword as keyof typeof technologyColors] || 0x667eea

    const particles = createTextParticles(
      keyword,
      color,
      0,  // 水平居中
      1 - idx * 1.1  // 向下移动开始位置
    )
    particles.forEach(particle => {
      textParticleScene.add(particle)
    })
  })

  if (!animationId1) {
    animateTextParticle()
  }
}

// 动画效果
const animateTextParticle = () => {
  animationId1 = requestAnimationFrame(animateTextParticle)

  textParticleScene.children.forEach((child) => {
    if (child instanceof THREE.Mesh && child.userData.originalPosition) {
      const time = Date.now() * 0.001
      const originalPos = child.userData.originalPosition

      child.position.x = originalPos.x + Math.sin(time + child.position.x) * 0.08
      child.position.y = originalPos.y + Math.cos(time + child.position.y) * 0.08
      child.position.z = originalPos.z + Math.sin(time * 0.5) * 0.05
      child.rotation.x += 0.03
      child.rotation.y += 0.03
    }
  })

  textParticleScene.rotation.y += 0.008
  textParticleRenderer?.render(textParticleScene, textParticleCamera)
}

onMounted(async () => {
  await nextTick()

  setTimeout(() => {
    initTextParticle()
  }, 100)

  // 监听窗口大小变化
  window.addEventListener('resize', () => {
    if (textParticleRenderer && textParticleScene) {
      const sectionElement = document.querySelector('.skills-section') as HTMLElement
      const width = sectionElement ? sectionElement.offsetWidth : window.innerWidth
      const height = sectionElement ? sectionElement.offsetHeight : window.innerHeight

      textParticleRenderer.setSize(width, height)
      textParticleCamera.aspect = width / height
      textParticleCamera.updateProjectionMatrix()
    }
  })
})

onUnmounted(() => {
  if (animationId1) cancelAnimationFrame(animationId1)
  textParticleRenderer?.dispose()
  window.removeEventListener('resize', () => {})
})
</script>

<style scoped>
.skills-section {
  width: 100vw;
  min-height: 100vh;
  background: #111;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 2rem 0;
  position: relative;
  overflow: hidden;
}

/* 背景 Canvas */
.background-canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1;
  pointer-events: none;
}

/* 内容层 */
.content-layer {
  position: relative;
  z-index: 2;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 2rem 0;
}

.section-title {
  font-size: 2.5rem;
  margin-bottom: 2rem;
  color: #667eea;
  text-align: center;
  text-shadow: 0 0 20px rgba(102, 126, 234, 0.5);
  position: relative;
  z-index: 3;
}

.skill-categories {
  display: flex;
  gap: 1rem;
  margin-bottom: 2rem;
  flex-wrap: wrap;
  justify-content: center;
  padding: 0 1rem;
  position: relative;
  z-index: 3;
}

.category-btn {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.8rem 1.5rem;
  background: rgba(255, 255, 255, 0.1);
  border: 2px solid rgba(255, 255, 255, 0.2);
  border-radius: 25px;
  color: #fff;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  backdrop-filter: blur(10px);
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
}

.category-btn:hover {
  background: rgba(102, 126, 234, 0.3);
  border-color: rgba(102, 126, 234, 0.6);
  color: #fff;
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(102, 126, 234, 0.4);
}

.category-btn.active {
  background: linear-gradient(135deg, #667eea, #42b883);
  border-color: transparent;
  color: #fff;
  box-shadow: 0 6px 25px rgba(102, 126, 234, 0.5);
}

.category-icon {
  font-size: 1.2rem;
}

@media (max-width: 768px) {
  .section-title {
    font-size: 2rem;
  }
  .skill-categories {
    gap: 0.5rem;
  }
  .category-btn {
    padding: 0.6rem 1rem;
    font-size: 0.9rem;
  }
  .category-icon {
    font-size: 1rem;
  }
}

@media (max-width: 480px) {
  .section-title {
    font-size: 2rem;
  }
  .skill-categories {
    flex-direction: column;
    align-items: center;
  }
  .category-btn {
    width: 200px;
    justify-content: center;
  }
}
</style>