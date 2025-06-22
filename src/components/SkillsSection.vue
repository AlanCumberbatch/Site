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

// 动画状态
let isTransitioning = false
let transitionStartTime = 0
let oldParticles: THREE.Mesh[] = []
let newParticles: THREE.Mesh[] = []
let oldTargetPositions: THREE.Vector3[] = []
let newStartPositions: THREE.Vector3[] = []

// 技能分类
const skillCategories = [
  { key: 'frontend', name: '前端', icon: '💻' },
  { key: '3d', name: '3D & 可视化', icon: '🌐' },
  { key: 'language', name: '开发语言', icon: '📝' },
  { key: 'backend', name: '后端 & 数据库', icon: '🗄️' },
  { key: 'tools', name: '工具 & 部署', icon: '🛠️' }
]

// 各分类的技能关键词
const skillKeywordsByCategory = {
  frontend: ['REACT', 'VUE.JS', 'HTML5', 'CSS3', 'SASS', 'WEBPACK', 'VITE'],
  '3d': ['THREE.JS', 'WEBGL', 'CESIUMJS', 'D3.JS', 'CANVAS', 'WEBGPU'],
  language: ['JAVASCRIPT', 'TYPESCRIPT', 'C++', 'PYTHON', 'JAVA'],
  backend: ['NODE.JS', 'EXPRESS', 'MONGODB', 'POSTGRESQL', 'REDIS', 'REST API'],
  tools: ['GIT', 'AWS', 'DOCKER', 'LINUX', 'NPM', 'YARN']
}

// 各技术的官方颜色
const technologyColors = {
  'REACT': 0x61dafb,
  'VUE.JS': 0x42b883,
  'HTML5': 0xe34c26,
  'CSS3': 0x1572b6,
  'SASS': 0xcc6699,
  'WEBPACK': 0x8dd6f9,
  'VITE': 0x646cff,
  'THREE.JS': 0x667eea,
  'WEBGL': 0x990000,
  'CESIUMJS': 0x6cbd45,
  'D3.JS': 0xff7f0e,
  'CANVAS': 0xffffff,
  'WEBGPU': 0x00aaff,
  'JAVASCRIPT': 0xf7df1e,
  'TYPESCRIPT': 0x3178c6,
  'C++': 0x00599c,
  'PYTHON': 0x3776ab,
  'JAVA': 0xed8b00,
  'NODE.JS': 0x3c873a,
  'EXPRESS': 0x000000,
  'MONGODB': 0x4db33d,
  'POSTGRESQL': 0x336791,
  'REDIS': 0xdc382d,
  'REST API': 0x00d4aa,
  'GIT': 0xf05032,
  'AWS': 0xff9900,
  'DOCKER': 0x2496ed,
  'LINUX': 0xfcc624,
  'NPM': 0xcb3837,
  'YARN': 0x2c8ebb
}

const activeCategory = ref('frontend')

const switchCategory = (categoryKey: string) => {
  if (isTransitioning) return // 防止重复点击

  activeCategory.value = categoryKey
  startTransition()
}

const getActiveCategoryName = () => {
  const category = skillCategories.find(cat => cat.key === activeCategory.value)
  return category ? category.name : ''
}

// 文字粒子化
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
          originalPosition: mesh.position.clone(),
          targetPosition: mesh.position.clone(),
          startPosition: mesh.position.clone()
        }

        particles.push(mesh)
      }
    }
  }
  return particles
}

// 开始过渡动画
const startTransition = () => {
  isTransitioning = true
  transitionStartTime = Date.now()

  // 保存当前粒子，但不从场景中移除
  oldParticles = [...textParticleScene.children] as THREE.Mesh[]
  oldTargetPositions = oldParticles.map(particle => {
    const randomAngle = Math.random() * Math.PI * 2
    const randomRadius = 10 + Math.random() * 20
    return new THREE.Vector3(
      Math.cos(randomAngle) * randomRadius,
      Math.sin(randomAngle) * randomRadius,
      (Math.random() - 0.5) * 10
    )
  })

  // 创建新粒子，但不清除旧粒子
  const currentKeywords = skillKeywordsByCategory[activeCategory.value as keyof typeof skillKeywordsByCategory]
  newParticles = []

  currentKeywords.forEach((keyword, idx) => {
    const color = technologyColors[keyword as keyof typeof technologyColors] || 0x667eea
    const particles = createTextParticles(
      keyword,
      color,
      0,
      1 - idx * 1.1
    )

    particles.forEach(particle => {
      // 设置起始位置为随机爆炸位置
      const randomAngle = Math.random() * Math.PI * 2
      const randomRadius = 15 + Math.random() * 25
      particle.position.set(
        Math.cos(randomAngle) * randomRadius,
        Math.sin(randomAngle) * randomRadius,
        (Math.random() - 0.5) * 15
      )

      particle.userData.startPosition = particle.position.clone()
      particle.userData.targetPosition = particle.userData.originalPosition.clone()

      // 初始透明度为0
      particle.material.opacity = 0

      newParticles.push(particle)
      textParticleScene.add(particle)
    })
  })

  newStartPositions = newParticles.map(particle => particle.position.clone())
}

const initTextParticle = () => {
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

  currentKeywords.forEach((keyword, idx) => {
    const color = technologyColors[keyword as keyof typeof technologyColors] || 0x667eea
    const particles = createTextParticles(
      keyword,
      color,
      0,
      1 - idx * 1.1
    )
    particles.forEach(particle => {
      textParticleScene.add(particle)
    })
  })

  if (!animationId1) {
    animateTextParticle()
  }
}

// 增强的动画效果
const animateTextParticle = () => {
  animationId1 = requestAnimationFrame(animateTextParticle)

  const currentTime = Date.now()

  if (isTransitioning) {
    const elapsed = currentTime - transitionStartTime
    const duration = 1500 // 1.5秒过渡时间
    const progress = Math.min(elapsed / duration, 1)

    // 使用缓动函数
    const easeOut = 1 - Math.pow(1 - progress, 3)
    const easeIn = Math.pow(progress, 3)

    // 动画旧粒子爆炸消散
    oldParticles.forEach((particle, idx) => {
      if (particle && oldTargetPositions[idx]) {
        const startPos = particle.userData.originalPosition
        const targetPos = oldTargetPositions[idx]

        // 旧粒子向随机方向爆炸
        particle.position.lerpVectors(startPos, targetPos, easeOut)

        // 旧粒子逐渐消失
        particle.material.opacity = 0.8 * (1 - easeOut)

        // 添加旋转效果
        particle.rotation.x += 0.1
        particle.rotation.y += 0.1
      }
    })

    // 动画新粒子重组
    newParticles.forEach((particle, idx) => {
      if (particle && newStartPositions[idx]) {
        const startPos = newStartPositions[idx]
        const targetPos = particle.userData.targetPosition

        // 新粒子从随机位置向目标位置聚集
        particle.position.lerpVectors(startPos, targetPos, easeIn)

        // 新粒子逐渐出现
        particle.material.opacity = 0.8 * easeIn

        // 添加旋转效果
        particle.rotation.x += 0.1
        particle.rotation.y += 0.1
      }
    })

    // 过渡完成
    if (progress >= 1) {
      isTransitioning = false

      // 移除旧粒子
      oldParticles.forEach(particle => {
        textParticleScene.remove(particle)
        particle.geometry.dispose()
        particle.material.dispose()
      })

      oldParticles = []
      newParticles = []
      oldTargetPositions = []
      newStartPositions = []
    }
  } else {
    // 正常动画
    textParticleScene.children.forEach((child) => {
      if (child instanceof THREE.Mesh && child.userData.originalPosition) {
        const time = currentTime * 0.001
        const originalPos = child.userData.originalPosition

        child.position.x = originalPos.x + Math.sin(time + child.position.x) * 0.08
        child.position.y = originalPos.y + Math.cos(time + child.position.y) * 0.08
        child.position.z = originalPos.z + Math.sin(time * 0.5) * 0.05
        child.rotation.x += 0.03
        child.rotation.y += 0.03
      }
    })
  }

  textParticleScene.rotation.y += 0.008
  textParticleRenderer?.render(textParticleScene, textParticleCamera)
}

onMounted(async () => {
  await nextTick()

  setTimeout(() => {
    initTextParticle()
  }, 100)

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