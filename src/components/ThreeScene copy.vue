<template>
  <div ref="containerRef" class="three-container">
    <div class="content-overlay">
      <div class="hero-section">
        <h1 class="title">欢迎来到我的个人网站</h1>
        <p class="subtitle">前端开发者 | 3D爱好者 | 创意探索者</p>
        <div class="cta-buttons">
          <button class="btn primary" @click="scrollToSection('about')">了解更多</button>
          <button class="btn secondary" @click="scrollToSection('projects')">查看作品</button>
        </div>
      </div>

      <div class="sections">
        <!-- 关于我 -->
        <section id="about" class="section">
          <h2>关于我</h2>
          <div class="about-content">
            <div class="about-text">
              <p>我是一名充满激情的前端开发者，专注于创建美观且功能强大的Web应用。</p>
              <p>热爱新技术，特别是Three.js、Vue.js和现代Web技术栈。</p>
            </div>
            <div class="skills">
              <h3>技能专长</h3>
              <div class="skill-tags">
                <span class="skill-tag">Vue.js</span>
                <span class="skill-tag">Three.js</span>
                <span class="skill-tag">TypeScript</span>
                <span class="skill-tag">React</span>
                <span class="skill-tag">Node.js</span>
                <span class="skill-tag">WebGL</span>
              </div>
            </div>
          </div>
        </section>

        <!-- 项目展示 -->
        <section id="projects" class="section">
          <h2>项目展示</h2>
          <div class="projects-grid">
            <div class="project-card" v-for="project in projects" :key="project.id">
              <div class="project-image">
                <div class="project-placeholder">{{ project.icon }}</div>
              </div>
              <h3>{{ project.title }}</h3>
              <p>{{ project.description }}</p>
              <div class="project-tech">
                <span v-for="tech in project.technologies" :key="tech" class="tech-tag">{{ tech }}</span>
              </div>
            </div>
          </div>
        </section>

        <!-- 联系方式 -->
        <section id="contact" class="section">
          <h2>联系我</h2>
          <div class="contact-content">
            <div class="contact-info">
              <div class="contact-item">
                <span class="contact-icon">📧</span>
                <span>email@example.com</span>
              </div>
              <div class="contact-item">
                <span class="contact-icon">📱</span>
                <span>+86 138-0000-0000</span>
              </div>
              <div class="contact-item">
                <span class="contact-icon">📍</span>
                <span>中国，北京</span>
              </div>
            </div>
            <div class="social-links">
              <a href="#" class="social-link">GitHub</a>
              <a href="#" class="social-link">LinkedIn</a>
              <a href="#" class="social-link">Twitter</a>
            </div>
          </div>
        </section>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { onMounted, ref, onUnmounted } from 'vue'
import * as THREE from 'three'

const containerRef = ref<HTMLElement | null>(null)
let scene: THREE.Scene
let camera: THREE.PerspectiveCamera
let renderer: THREE.WebGLRenderer
let particleGroup: THREE.Group
let raycaster: THREE.Raycaster
let mouse: THREE.Vector2
let animationId: number
let hoveredParticle: THREE.Mesh | null = null
let particles: THREE.Mesh[] = []
let lights: THREE.Light[] = []

// 项目数据
const projects = ref([
  {
    id: 1,
    title: '3D产品展示',
    description: '基于Three.js的交互式3D产品展示平台',
    icon: '🎨',
    technologies: ['Three.js', 'Vue.js', 'WebGL']
  },
  {
    id: 2,
    title: '数据可视化',
    description: '实时数据可视化仪表板',
    icon: '📊',
    technologies: ['D3.js', 'React', 'TypeScript']
  },
  {
    id: 3,
    title: '游戏引擎',
    description: '轻量级Web游戏引擎',
    icon: '🎮',
    technologies: ['Three.js', 'Cannon.js', 'WebGL']
  }
])

const initThree = () => {
  // 创建场景
  scene = new THREE.Scene()

  // 创建相机
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
  camera.position.z = 5

  // 创建渲染器
  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setClearColor(0x000000, 0.1)
  renderer.shadowMap.enabled = true
  renderer.shadowMap.type = THREE.PCFSoftShadowMap
  containerRef.value?.appendChild(renderer.domElement)

  // 初始化鼠标和射线检测器
  mouse = new THREE.Vector2()
  raycaster = new THREE.Raycaster()

  // 创建光源系统
  createLightingSystem()

  // 创建粒子系统
  createParticleSystem()

  // 添加鼠标事件监听
  addMouseListeners()

  animate()
}

const createLightingSystem = () => {
  // 1. 环境光 - 大幅增加亮度
  const ambientLight = new THREE.AmbientLight(0xffffff, 1.2) // 改为白色，强度1.2
  scene.add(ambientLight)
  lights.push(ambientLight)

  // 2. 主光源 - 增加强度
  const mainLight = new THREE.PointLight(0xffffff, 1.5, 100) // 强度从1增加到1.5
  mainLight.position.set(10, 10, 10)
  mainLight.castShadow = true
  mainLight.shadow.mapSize.width = 2048
  mainLight.shadow.mapSize.height = 2048
  scene.add(mainLight)
  lights.push(mainLight)

  // 3. 彩色点光源 - 增加强度
  const blueLight = new THREE.PointLight(0x0066ff, 1.2, 50) // 强度从0.8增加到1.2
  blueLight.position.set(-8, 5, 8)
  scene.add(blueLight)
  lights.push(blueLight)

  // 4. 彩色点光源 - 增加强度
  const purpleLight = new THREE.PointLight(0x6600ff, 1.2, 50) // 强度从0.8增加到1.2
  purpleLight.position.set(8, -5, -8)
  scene.add(purpleLight)
  lights.push(purpleLight)

  // 5. 彩色点光源 - 增加强度
  const orangeLight = new THREE.PointLight(0xff6600, 1.0, 40) // 强度从0.6增加到1.0
  orangeLight.position.set(0, -8, 0)
  scene.add(orangeLight)
  lights.push(orangeLight)

  // 6. 聚光灯 - 增加强度
  const spotLight = new THREE.SpotLight(0xffffff, 0.8, 100, Math.PI / 6, 0.5, 1) // 强度从0.5增加到0.8
  spotLight.position.set(0, 15, 0)
  spotLight.target.position.set(0, 0, 0)
  scene.add(spotLight)
  scene.add(spotLight.target)
  lights.push(spotLight)

  // 7. 方向光 - 增加强度
  const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8) // 强度从0.4增加到0.8
  directionalLight.position.set(5, 5, 5)
  directionalLight.castShadow = true
  scene.add(directionalLight)
  lights.push(directionalLight)
}

const createParticleSystem = () => {
  particleGroup = new THREE.Group()
  scene.add(particleGroup)

  const particleCount = 100

  for (let i = 0; i < particleCount; i++) {
    // 创建球形分布的粒子
    const radius = 6
    const theta = Math.random() * Math.PI * 2
    const phi = Math.acos(Math.random() * 2 - 1)

    const x = radius * Math.sin(phi) * Math.cos(theta)
    const y = radius * Math.sin(phi) * Math.sin(theta)
    const z = radius * Math.cos(phi)

    // 创建小球体几何体
    const geometry = new THREE.SphereGeometry(0.1, 8, 8)

    // 创建材质 - 增加发光效果
    const material = new THREE.MeshPhongMaterial({
      color: new THREE.Color(
        0.3 + Math.random() * 0.3,
        0.5 + Math.random() * 0.3,
        0.8 + Math.random() * 0.2
      ),
      transparent: true,
      opacity: 0.9, // 增加不透明度
      shininess: 100,
      emissive: new THREE.Color(0x222222) // 增加基础发光
    })

    const particle = new THREE.Mesh(geometry, material)
    particle.position.set(x, y, z)
    particle.castShadow = true
    particle.receiveShadow = true

    // 存储原始颜色和大小
    particle.userData = {
      originalColor: material.color.clone(),
      originalScale: 1,
      isHovered: false
    }

    particles.push(particle)
    particleGroup.add(particle)
  }
}

const addMouseListeners = () => {
  const canvas = renderer.domElement

  canvas.addEventListener('mousemove', onMouseMove)
  canvas.addEventListener('mouseenter', onMouseEnter)
  canvas.addEventListener('mouseleave', onMouseLeave)
}

const onMouseMove = (event: MouseEvent) => {
  const rect = renderer.domElement.getBoundingClientRect()
  mouse.x = ((event.clientX - rect.left) / rect.width) * 2 - 1
  mouse.y = -((event.clientY - rect.top) / rect.height) * 2 + 1

  // 射线检测
  raycaster.setFromCamera(mouse, camera)
  const intersects = raycaster.intersectObjects(particles, false)

  // 重置之前悬停的粒子
  if (hoveredParticle && hoveredParticle.userData.isHovered) {
    resetParticle(hoveredParticle)
    hoveredParticle = null
  }

  if (intersects.length > 0) {
    const intersectedParticle = intersects[0].object as THREE.Mesh
    if (intersectedParticle !== hoveredParticle) {
      hoveredParticle = intersectedParticle
      hoverParticle(intersectedParticle)
    }
  } else {
    hoveredParticle = null
  }
}

const onMouseEnter = () => {
  // 鼠标进入画布
}

const onMouseLeave = () => {
  // 鼠标离开画布，重置所有悬停状态
  if (hoveredParticle) {
    resetParticle(hoveredParticle)
    hoveredParticle = null
  }
}

const hoverParticle = (particle: THREE.Mesh) => {
  const material = particle.material as THREE.MeshPhongMaterial
  material.color.setHex(0xff6600) // 橙色
  material.emissive.setHex(0x666666) // 增加发光效果
  particle.scale.setScalar(2) // 放大2倍
  particle.userData.isHovered = true
}

const resetParticle = (particle: THREE.Mesh) => {
  const material = particle.material as THREE.MeshPhongMaterial
  material.color.copy(particle.userData.originalColor)
  material.emissive.setHex(0x222222) // 恢复基础发光
  particle.scale.setScalar(particle.userData.originalScale)
  particle.userData.isHovered = false
}

const animate = () => {
  animationId = requestAnimationFrame(animate)

  const time = Date.now() * 0.001

  // 光源动画
  lights.forEach((light, index) => {
    if (light instanceof THREE.PointLight) {
      // 让点光源缓慢移动
      const speed = 0.5 + index * 0.2
      light.position.x = Math.sin(time * speed) * 10
      light.position.y = Math.cos(time * speed * 0.7) * 8
      light.position.z = Math.sin(time * speed * 0.5) * 10
    }
  })

  if (particleGroup) {
    // 缓慢旋转整个粒子系统
    particleGroup.rotation.x += 0.001
    particleGroup.rotation.y += 0.002

    // 让粒子有轻微的呼吸效果
    particles.forEach((particle, index) => {
      if (!particle.userData.isHovered) {
        const breathingEffect = Math.sin(time + index * 0.1) * 0.1 + 1
        particle.scale.setScalar(breathingEffect)
      }
    })
  }

  renderer.render(scene, camera)
}

const handleResize = () => {
  if (camera && renderer) {
    camera.aspect = window.innerWidth / window.innerHeight
    camera.updateProjectionMatrix()
    renderer.setSize(window.innerWidth, window.innerHeight)
  }
}

const scrollToSection = (sectionId: string) => {
  const element = document.getElementById(sectionId)
  if (element) {
    element.scrollIntoView({ behavior: 'smooth' })
  }
}

onMounted(() => {
  initThree()
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  if (animationId) {
    cancelAnimationFrame(animationId)
  }
  window.removeEventListener('resize', handleResize)

  // 移除鼠标事件监听
  if (renderer) {
    const canvas = renderer.domElement
    canvas.removeEventListener('mousemove', onMouseMove)
    canvas.removeEventListener('mouseenter', onMouseEnter)
    canvas.removeEventListener('mouseleave', onMouseLeave)
    renderer.dispose()
  }
})
</script>

<style scoped>
.three-container {
  width: 100vw;
  height: 100vh;
  overflow-x: hidden;
  position: relative;
}

.content-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 10;
  overflow-y: auto;
  scroll-behavior: smooth;
  /* 隐藏滚动条 */
  -ms-overflow-style: none;  /* IE and Edge */
  scrollbar-width: none;  /* Firefox */
}

.content-overlay::-webkit-scrollbar {
  display: none;  /* Chrome, Safari and Opera */
}

.hero-section {
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  text-align: center;
  padding: 0 2rem;
  background: linear-gradient(135deg, rgba(0,0,0,0.7) 0%, rgba(0,0,0,0.3) 100%);
}

.title {
  font-size: 4rem;
  font-weight: 700;
  margin-bottom: 1rem;
  background: linear-gradient(45deg, #667eea 0%, #764ba2 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  animation: fadeInUp 1s ease-out;
}

.subtitle {
  font-size: 1.5rem;
  color: rgba(255, 255, 255, 0.8);
  margin-bottom: 3rem;
  animation: fadeInUp 1s ease-out 0.3s both;
}

.cta-buttons {
  display: flex;
  gap: 1rem;
  animation: fadeInUp 1s ease-out 0.6s both;
}

.btn {
  padding: 1rem 2rem;
  border: none;
  border-radius: 50px;
  font-size: 1.1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  text-decoration: none;
  display: inline-block;
}

.btn.primary {
  background: linear-gradient(45deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.btn.secondary {
  background: transparent;
  color: white;
  border: 2px solid rgba(255, 255, 255, 0.3);
}

.btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
}

.sections {
  background: rgba(0, 0, 0, 0.8);
  backdrop-filter: blur(10px);
}

.section {
  padding: 5rem 2rem;
  max-width: 1200px;
  margin: 0 auto;
}

.section h2 {
  font-size: 2.5rem;
  margin-bottom: 3rem;
  text-align: center;
  color: #667eea;
}

.about-content {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 3rem;
  align-items: start;
}

.about-text p {
  font-size: 1.2rem;
  line-height: 1.8;
  margin-bottom: 1rem;
  color: rgba(255, 255, 255, 0.9);
}

.skills h3 {
  font-size: 1.5rem;
  margin-bottom: 1.5rem;
  color: #667eea;
}

.skill-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.skill-tag {
  padding: 0.5rem 1rem;
  background: rgba(102, 126, 234, 0.2);
  border: 1px solid rgba(102, 126, 234, 0.3);
  border-radius: 25px;
  color: #667eea;
  font-size: 0.9rem;
}

.projects-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 2rem;
}

.project-card {
  background: rgba(255, 255, 255, 0.05);
  border-radius: 15px;
  padding: 2rem;
  border: 1px solid rgba(255, 255, 255, 0.1);
  transition: all 0.3s ease;
}

.project-card:hover {
  transform: translateY(-5px);
  background: rgba(255, 255, 255, 0.1);
}

.project-image {
  margin-bottom: 1rem;
}

.project-placeholder {
  width: 60px;
  height: 60px;
  background: linear-gradient(45deg, #667eea 0%, #764ba2 100%);
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 2rem;
}

.project-card h3 {
  font-size: 1.3rem;
  margin-bottom: 1rem;
  color: #667eea;
}

.project-card p {
  color: rgba(255, 255, 255, 0.8);
  margin-bottom: 1rem;
  line-height: 1.6;
}

.project-tech {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.tech-tag {
  padding: 0.3rem 0.8rem;
  background: rgba(102, 126, 234, 0.1);
  border-radius: 15px;
  font-size: 0.8rem;
  color: #667eea;
}

.contact-content {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 3rem;
  align-items: center;
}

.contact-info {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.contact-item {
  display: flex;
  align-items: center;
  gap: 1rem;
  font-size: 1.1rem;
  color: rgba(255, 255, 255, 0.9);
}

.contact-icon {
  font-size: 1.5rem;
}

.social-links {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.social-link {
  padding: 1rem 2rem;
  background: rgba(102, 126, 234, 0.1);
  border: 1px solid rgba(102, 126, 234, 0.3);
  border-radius: 10px;
  color: #667eea;
  text-decoration: none;
  text-align: center;
  transition: all 0.3s ease;
}

.social-link:hover {
  background: rgba(102, 126, 234, 0.2);
  transform: translateX(5px);
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
  .title {
    font-size: 2.5rem;
  }

  .subtitle {
    font-size: 1.2rem;
  }

  .about-content,
  .contact-content {
    grid-template-columns: 1fr;
  }

  .cta-buttons {
    flex-direction: column;
  }

  .section {
    padding: 3rem 1rem;
  }
}
</style>