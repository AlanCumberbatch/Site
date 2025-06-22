<template>
  <div ref="containerRef" class="text3d-container"></div>
</template>

<script lang="ts" setup>
import { onMounted, ref, onUnmounted } from 'vue'
import * as THREE from 'three'

const containerRef = ref<HTMLElement | null>(null)
let scene: THREE.Scene
let camera: THREE.PerspectiveCamera
let renderer: THREE.WebGLRenderer
let textMesh: THREE.Mesh
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

  createText()
  animate()
}

const createText = () => {
  // 使用基础几何体创建3D文字效果
  const text = 'Hello World'
  const canvas = document.createElement('canvas')
  const context = canvas.getContext('2d')

  if (!context) return

  canvas.width = 512
  canvas.height = 128

  context.fillStyle = '#ffffff'
  context.font = 'bold 64px Arial'
  context.fillText(text, 10, 80)

  const texture = new THREE.CanvasTexture(canvas)

  // 创建平面几何体
  const geometry = new THREE.PlaneGeometry(4, 1)
  const material = new THREE.MeshBasicMaterial({
    map: texture,
    transparent: true,
    opacity: 0.8
  })

  textMesh = new THREE.Mesh(geometry, material)
  scene.add(textMesh)

  // 添加3D立体效果的文字
  create3DText()
}

const create3DText = () => {
  // 创建多个平面层叠形成3D效果
  const text = '3D TEXT'
  const layers = 10
  const layerHeight = 0.1

  for (let i = 0; i < layers; i++) {
    const canvas = document.createElement('canvas')
    const context = canvas.getContext('2d')

    if (!context) continue

    canvas.width = 512
    canvas.height = 128

    context.fillStyle = `hsl(${200 + i * 10}, 70%, 50%)`
    context.font = 'bold 64px Arial'
    context.fillText(text, 10, 80)

    const texture = new THREE.CanvasTexture(canvas)
    const geometry = new THREE.PlaneGeometry(4, 1)
    const material = new THREE.MeshBasicMaterial({
      map: texture,
      transparent: true,
      opacity: 0.8
    })

    const layer = new THREE.Mesh(geometry, material)
    layer.position.z = i * layerHeight
    scene.add(layer)
  }
}

const animate = () => {
  animationId = requestAnimationFrame(animate)

  if (textMesh) {
    textMesh.rotation.y += 0.01
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
.text3d-container {
  width: 100vw;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
  z-index: -1;
}
</style>