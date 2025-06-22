<template>
  <div ref="containerRef" class="model-container"></div>
</template>

<script lang="ts" setup>
import { onMounted, ref, onUnmounted } from 'vue'
import * as THREE from 'three'

const containerRef = ref<HTMLElement | null>(null)
let scene: THREE.Scene
let camera: THREE.PerspectiveCamera
let renderer: THREE.WebGLRenderer
let model: THREE.Group
let animationId: number

const initThree = () => {
  scene = new THREE.Scene()
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
  camera.position.z = 5

  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setClearColor(0x000000, 0.1)
  containerRef.value?.appendChild(renderer.domElement)

  // 添加光源
  const ambientLight = new THREE.AmbientLight(0x404040, 0.6)
  scene.add(ambientLight)

  const directionalLight = new THREE.DirectionalLight(0xffffff, 1)
  directionalLight.position.set(10, 10, 5)
  scene.add(directionalLight)

  createModel()
  animate()
}

const createModel = () => {
  // 创建一个简单的几何体组合作为示例模型
  model = new THREE.Group()

  // 主体
  const bodyGeometry = new THREE.CylinderGeometry(0.5, 0.5, 2, 8)
  const bodyMaterial = new THREE.MeshPhongMaterial({ color: 0x888888 })
  const body = new THREE.Mesh(bodyGeometry, bodyMaterial)
  model.add(body)

  // 头部
  const headGeometry = new THREE.SphereGeometry(0.3, 8, 8)
  const headMaterial = new THREE.MeshPhongMaterial({ color: 0xcccccc })
  const head = new THREE.Mesh(headGeometry, headMaterial)
  head.position.y = 1.5
  model.add(head)

  // 手臂
  const armGeometry = new THREE.CylinderGeometry(0.1, 0.1, 1, 8)
  const armMaterial = new THREE.MeshPhongMaterial({ color: 0x666666 })

  const leftArm = new THREE.Mesh(armGeometry, armMaterial)
  leftArm.position.set(-0.8, 0.5, 0)
  leftArm.rotation.z = Math.PI / 4
  model.add(leftArm)

  const rightArm = new THREE.Mesh(armGeometry, armMaterial)
  rightArm.position.set(0.8, 0.5, 0)
  rightArm.rotation.z = -Math.PI / 4
  model.add(rightArm)

  scene.add(model)
}

const animate = () => {
  animationId = requestAnimationFrame(animate)

  if (model) {
    model.rotation.y += 0.01
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

onMounted(() => {
  initThree()
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  if (animationId) {
    cancelAnimationFrame(animationId)
  }
  window.removeEventListener('resize', handleResize)
  if (renderer) {
    renderer.dispose()
  }
})
</script>

<style scoped>
.model-container {
  width: 100vw;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
  z-index: -1;
}
</style>