<template>
  <div ref="containerRef" class="cube-container"></div>
</template>

<script lang="ts" setup>
import { onMounted, ref, onUnmounted } from 'vue'
import * as THREE from 'three'

const containerRef = ref<HTMLElement | null>(null)
let scene: THREE.Scene
let camera: THREE.PerspectiveCamera
let renderer: THREE.WebGLRenderer
let cube: THREE.Mesh
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

  createCube()
  animate()
}

const createCube = () => {
  const geometry = new THREE.BoxGeometry(1, 1, 1)

  // 创建六面不同颜色的材质
  const materials = [
    new THREE.MeshPhongMaterial({ color: 0xff0000 }), // 红
    new THREE.MeshPhongMaterial({ color: 0x00ff00 }), // 绿
    new THREE.MeshPhongMaterial({ color: 0x0000ff }), // 蓝
    new THREE.MeshPhongMaterial({ color: 0xffff00 }), // 黄
    new THREE.MeshPhongMaterial({ color: 0xff00ff }), // 紫
    new THREE.MeshPhongMaterial({ color: 0x00ffff })  // 青
  ]

  cube = new THREE.Mesh(geometry, materials)
  scene.add(cube)
}

const animate = () => {
  animationId = requestAnimationFrame(animate)

  if (cube) {
    cube.rotation.x += 0.01
    cube.rotation.y += 0.01
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
.cube-container {
  width: 100vw;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
  z-index: -1;
}
</style>