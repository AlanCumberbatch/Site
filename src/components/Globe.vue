<template>
  <div ref="containerRef" class="globe-container"></div>
</template>

<script lang="ts" setup>
import { onMounted, ref, onUnmounted } from 'vue'
import * as THREE from 'three'

const containerRef = ref<HTMLElement | null>(null)
let scene: THREE.Scene
let camera: THREE.PerspectiveCamera
let renderer: THREE.WebGLRenderer
let globe: THREE.Mesh
let animationId: number

const initThree = () => {
  scene = new THREE.Scene()
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
  camera.position.z = 3

  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setClearColor(0x000000, 0.1)
  containerRef.value?.appendChild(renderer.domElement)

  // 添加光源
  const ambientLight = new THREE.AmbientLight(0x404040, 0.4)
  scene.add(ambientLight)

  const directionalLight = new THREE.DirectionalLight(0xffffff, 1)
  directionalLight.position.set(5, 3, 5)
  scene.add(directionalLight)

  createGlobe()
  animate()
}

const createGlobe = () => {
  const geometry = new THREE.SphereGeometry(1, 32, 32)

  // 创建地球材质
  const material = new THREE.MeshPhongMaterial({
    color: 0x0066cc,
    transparent: true,
    opacity: 0.8,
    shininess: 100
  })

  globe = new THREE.Mesh(geometry, material)
  scene.add(globe)

  // 添加大气层效果
  const atmosphereGeometry = new THREE.SphereGeometry(1.1, 32, 32)
  const atmosphereMaterial = new THREE.MeshBasicMaterial({
    color: 0x0066cc,
    transparent: true,
    opacity: 0.1
  })
  const atmosphere = new THREE.Mesh(atmosphereGeometry, atmosphereMaterial)
  scene.add(atmosphere)
}

const animate = () => {
  animationId = requestAnimationFrame(animate)

  if (globe) {
    globe.rotation.y += 0.005
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
.globe-container {
  width: 100vw;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
  z-index: -1;
}
</style>