<template>
  <div ref="containerRef" class="starfield-container"></div>
</template>

<script lang="ts" setup>
import { onMounted, ref, onUnmounted } from 'vue'
import * as THREE from 'three'

const containerRef = ref<HTMLElement | null>(null)
let scene: THREE.Scene
let camera: THREE.PerspectiveCamera
let renderer: THREE.WebGLRenderer
let stars: THREE.Points
let animationId: number

const initThree = () => {
  scene = new THREE.Scene()
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
  camera.position.z = 5

  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setClearColor(0x000011, 0.1)
  containerRef.value?.appendChild(renderer.domElement)

  createStarfield()
  animate()
}

const createStarfield = () => {
  const starCount = 3000
  const positions = new Float32Array(starCount * 3)
  const sizes = new Float32Array(starCount)

  for (let i = 0; i < starCount * 3; i += 3) {
    positions[i] = (Math.random() - 0.5) * 2000
    positions[i + 1] = (Math.random() - 0.5) * 2000
    positions[i + 2] = (Math.random() - 0.5) * 2000

    sizes[i / 3] = Math.random() * 2
  }

  const geometry = new THREE.BufferGeometry()
  geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))
  geometry.setAttribute('size', new THREE.BufferAttribute(sizes, 1))

  const material = new THREE.PointsMaterial({
    size: 1,
    color: 0xffffff,
    transparent: true,
    opacity: 0.8,
    sizeAttenuation: true
  })

  stars = new THREE.Points(geometry, material)
  scene.add(stars)
}

const animate = () => {
  animationId = requestAnimationFrame(animate)

  if (stars) {
    stars.rotation.y += 0.0005
    stars.rotation.x += 0.0002
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
.starfield-container {
  width: 100vw;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
  z-index: -1;
}
</style>