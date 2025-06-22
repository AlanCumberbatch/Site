<template>
  <section id="projects" class="projects-section">
    <h2 class="section-title">项目展示</h2>
    <div class="projects-grid">
      <div
        v-for="project in projects"
        :key="project.id"
        class="project-card"
        @click="openProject(project)"
      >
        <div class="project-3d-container" :ref="el => setProjectRef(el, project.id)">
          <!-- 3D模型将在这里渲染 -->
        </div>
        <div class="project-info">
          <h3>{{ project.title }}</h3>
          <p>{{ project.description }}</p>
          <div class="project-tech">
            <span v-for="tech in project.technologies" :key="tech" class="tech-tag">
              {{ tech }}
            </span>
          </div>
        </div>
      </div>
    </div>

    <!-- 项目详情模态框 -->
    <div v-if="selectedProject" class="project-modal" @click="closeProject">
      <div class="modal-content" @click.stop>
        <button class="close-btn" @click="closeProject">×</button>
        <div class="modal-3d-container" ref="modal3dRef">
          <!-- 3D交互Demo将在这里渲染 -->
        </div>
        <div class="modal-info">
          <h2>{{ selectedProject.title }}</h2>
          <p>{{ selectedProject.description }}</p>
          <div class="project-links">
            <a :href="selectedProject.demo" target="_blank" class="demo-link">查看Demo</a>
            <a :href="selectedProject.github" target="_blank" class="github-link">GitHub</a>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script lang="ts" setup>
import { onMounted, ref, onUnmounted } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'

const projects = ref([
  {
    id: 1,
    title: '3D产品展示',
    description: '基于Three.js的交互式3D产品展示平台',
    technologies: ['Three.js', 'Vue.js', 'WebGL'],
    demo: '#',
    github: '#',
    modelType: 'cube'
  },
  {
    id: 2,
    title: '数据可视化',
    description: '实时数据可视化仪表板',
    technologies: ['D3.js', 'React', 'TypeScript'],
    demo: '#',
    github: '#',
    modelType: 'sphere'
  },
  {
    id: 3,
    title: '游戏引擎',
    description: '轻量级Web游戏引擎',
    technologies: ['Three.js', 'Cannon.js', 'WebGL'],
    demo: '#',
    github: '#',
    modelType: 'torus'
  }
])

const selectedProject = ref(null)
const projectRefs = ref<Map<number, HTMLElement>>(new Map())
const modal3dRef = ref<HTMLElement | null>(null)
const renderers = ref<Map<number, THREE.WebGLRenderer>>(new Map())
const modalRenderer = ref<THREE.WebGLRenderer | null>(null)

const setProjectRef = (el: HTMLElement | null, projectId: number) => {
  if (el) {
    projectRefs.value.set(projectId, el)
    initProject3D(projectId, el)
  }
}

const initProject3D = (projectId: number, container: HTMLElement) => {
  const scene = new THREE.Scene()
  const camera = new THREE.PerspectiveCamera(75, 200 / 200, 0.1, 1000)
  camera.position.z = 3

  const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
  renderer.setSize(200, 200)
  renderer.setClearColor(0x000000, 0.1)
  container.appendChild(renderer.domElement)

  // 添加光源
  const ambientLight = new THREE.AmbientLight(0x404040, 0.6)
  scene.add(ambientLight)

  const directionalLight = new THREE.DirectionalLight(0xffffff, 1)
  directionalLight.position.set(10, 10, 5)
  scene.add(directionalLight)

  // 创建3D模型
  const project = projects.value.find(p => p.id === projectId)
  if (project) {
    createModel(scene, project.modelType)
  }

  renderers.value.set(projectId, renderer)

  // 动画循环
  const animate = () => {
    requestAnimationFrame(animate)

    // 旋转模型
    scene.children.forEach(child => {
      if (child instanceof THREE.Mesh) {
        child.rotation.x += 0.01
        child.rotation.y += 0.01
      }
    })

    renderer.render(scene, camera)
  }

  animate()
}

const createModel = (scene: THREE.Scene, modelType: string) => {
  let geometry: THREE.BufferGeometry
  let material: THREE.Material

  switch (modelType) {
    case 'cube':
      geometry = new THREE.BoxGeometry(1, 1, 1)
      material = new THREE.MeshPhongMaterial({
        color: 0x667eea,
        transparent: true,
        opacity: 0.8
      })
      break
    case 'sphere':
      geometry = new THREE.SphereGeometry(0.5, 16, 16)
      material = new THREE.MeshPhongMaterial({
        color: 0x764ba2,
        transparent: true,
        opacity: 0.8
      })
      break
    case 'torus':
      geometry = new THREE.TorusGeometry(0.5, 0.2, 8, 16)
      material = new THREE.MeshPhongMaterial({
        color: 0xff6600,
        transparent: true,
        opacity: 0.8
      })
      break
    default:
      geometry = new THREE.BoxGeometry(1, 1, 1)
      material = new THREE.MeshPhongMaterial({ color: 0x666666 })
  }

  const mesh = new THREE.Mesh(geometry, material)
  scene.add(mesh)
}

const openProject = (project: any) => {
  selectedProject.value = project
  setTimeout(() => {
    initModal3D(project)
  }, 100)
}

const closeProject = () => {
  selectedProject.value = null
  if (modalRenderer.value) {
    modalRenderer.value.dispose()
    modalRenderer.value = null
  }
}

const initModal3D = (project: any) => {
  if (!modal3dRef.value) return

  const scene = new THREE.Scene()
  const camera = new THREE.PerspectiveCamera(75, 400 / 300, 0.1, 1000)
  camera.position.z = 5

  const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
  renderer.setSize(400, 300)
  renderer.setClearColor(0x000000, 0.1)
  modal3dRef.value.appendChild(renderer.domElement)

  // 添加OrbitControls
  const controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true
  controls.dampingFactor = 0.05

  // 添加光源
  const ambientLight = new THREE.AmbientLight(0x404040, 0.6)
  scene.add(ambientLight)

  const directionalLight = new THREE.DirectionalLight(0xffffff, 1)
  directionalLight.position.set(10, 10, 5)
  scene.add(directionalLight)

  // 创建更复杂的模型
  createComplexModel(scene, project.modelType)

  modalRenderer.value = renderer

  // 动画循环
  const animate = () => {
    requestAnimationFrame(animate)
    controls.update()
    renderer.render(scene, camera)
  }

  animate()
}

const createComplexModel = (scene: THREE.Scene, modelType: string) => {
  // 创建更复杂的交互式模型
  const group = new THREE.Group()

  switch (modelType) {
    case 'cube':
      // 创建多个立方体
      for (let i = 0; i < 5; i++) {
        const geometry = new THREE.BoxGeometry(0.3, 0.3, 0.3)
        const material = new THREE.MeshPhongMaterial({
          color: new THREE.Color().setHSL(i / 5, 0.7, 0.5),
          transparent: true,
          opacity: 0.8
        })
        const mesh = new THREE.Mesh(geometry, material)
        mesh.position.set(
          (Math.random() - 0.5) * 2,
          (Math.random() - 0.5) * 2,
          (Math.random() - 0.5) * 2
        )
        group.add(mesh)
      }
      break
    case 'sphere':
      // 创建球体阵列
      for (let i = 0; i < 8; i++) {
        const geometry = new THREE.SphereGeometry(0.2, 8, 8)
        const material = new THREE.MeshPhongMaterial({
          color: new THREE.Color().setHSL(i / 8, 0.7, 0.5),
          transparent: true,
          opacity: 0.8
        })
        const mesh = new THREE.Mesh(geometry, material)
        const angle = (i / 8) * Math.PI * 2
        mesh.position.set(
          Math.cos(angle) * 1.5,
          Math.sin(angle) * 1.5,
          0
        )
        group.add(mesh)
      }
      break
    case 'torus':
      // 创建环形阵列
      for (let i = 0; i < 3; i++) {
        const geometry = new THREE.TorusGeometry(0.5 + i * 0.3, 0.1, 8, 16)
        const material = new THREE.MeshPhongMaterial({
          color: new THREE.Color().setHSL(i / 3, 0.7, 0.5),
          transparent: true,
          opacity: 0.6
        })
        const mesh = new THREE.Mesh(geometry, material)
        mesh.rotation.x = Math.PI / 2
        group.add(mesh)
      }
      break
  }

  scene.add(group)
}

onUnmounted(() => {
  // 清理所有渲染器
  renderers.value.forEach(renderer => {
    renderer.dispose()
  })
  if (modalRenderer.value) {
    modalRenderer.value.dispose()
  }
})
</script>

<style scoped>
.projects-section {
  padding: 5rem 2rem;
  background: rgba(0, 0, 0, 0.8);
  backdrop-filter: blur(10px);
}

.section-title {
  font-size: 2.5rem;
  margin-bottom: 3rem;
  text-align: center;
  color: #667eea;
}

.projects-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 2rem;
  max-width: 1200px;
  margin: 0 auto;
}

.project-card {
  background: rgba(255, 255, 255, 0.05);
  border-radius: 15px;
  padding: 2rem;
  border: 1px solid rgba(255, 255, 255, 0.1);
  transition: all 0.3s ease;
  cursor: pointer;
}

.project-card:hover {
  transform: translateY(-5px);
  background: rgba(255, 255, 255, 0.1);
}

.project-3d-container {
  width: 200px;
  height: 200px;
  margin: 0 auto 1rem;
  border-radius: 10px;
  overflow: hidden;
}

.project-info h3 {
  font-size: 1.3rem;
  margin-bottom: 1rem;
  color: #667eea;
}

.project-info p {
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

.project-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.9);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 15px;
  padding: 2rem;
  max-width: 800px;
  width: 90%;
  max-height: 80vh;
  overflow-y: auto;
  position: relative;
}

.close-btn {
  position: absolute;
  top: 1rem;
  right: 1rem;
  background: none;
  border: none;
  color: white;
  font-size: 2rem;
  cursor: pointer;
  z-index: 1001;
}

.modal-3d-container {
  width: 400px;
  height: 300px;
  margin: 0 auto 2rem;
  border-radius: 10px;
  overflow: hidden;
}

.modal-info h2 {
  color: #667eea;
  margin-bottom: 1rem;
}

.modal-info p {
  color: rgba(255, 255, 255, 0.8);
  margin-bottom: 2rem;
  line-height: 1.6;
}

.project-links {
  display: flex;
  gap: 1rem;
}

.demo-link, .github-link {
  padding: 0.8rem 1.5rem;
  border-radius: 25px;
  text-decoration: none;
  font-weight: 600;
  transition: all 0.3s ease;
}

.demo-link {
  background: linear-gradient(45deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.github-link {
  background: transparent;
  color: white;
  border: 2px solid rgba(255, 255, 255, 0.3);
}

.demo-link:hover, .github-link:hover {
  transform: translateY(-2px);
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
}

@media (max-width: 768px) {
  .projects-grid {
    grid-template-columns: 1fr;
  }

  .modal-content {
    width: 95%;
    padding: 1rem;
  }

  .modal-3d-container {
    width: 100%;
    height: 250px;
  }
}
</style>