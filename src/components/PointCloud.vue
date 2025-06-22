<template>
  <div ref="containerRef" class="pointcloud-container"></div>
</template>

<script lang="ts" setup>
import { onMounted, ref, onUnmounted } from 'vue'
import * as THREE from 'three'

const containerRef = ref<HTMLElement | null>(null)
let scene: THREE.Scene
let camera: THREE.PerspectiveCamera
let renderer: THREE.WebGLRenderer
let points: THREE.Points
let lines: THREE.LineSegments
let animationId: number
let time = 0

const initThree = () => {
  scene = new THREE.Scene()
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
  camera.position.z = 10

  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setClearColor(0x000000, 0.1)
  containerRef.value?.appendChild(renderer.domElement)

  createPointCloud()
  animate()
}

const createPointCloud = () => {
  const pointCount = 100
  const positions = new Float32Array(pointCount * 3)
  const colors = new Float32Array(pointCount * 3)

  // 创建点
  for (let i = 0; i < pointCount * 3; i += 3) {
    positions[i] = (Math.random() - 0.5) * 20
    positions[i + 1] = (Math.random() - 0.5) * 20
    positions[i + 2] = (Math.random() - 0.5) * 20

    colors[i] = Math.random()
    colors[i + 1] = Math.random()
    colors[i + 2] = Math.random()
  }

  // 创建点云
  const pointGeometry = new THREE.BufferGeometry()
  pointGeometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))
  pointGeometry.setAttribute('color', new THREE.BufferAttribute(colors, 3))

  const pointMaterial = new THREE.PointsMaterial({
    size: 0.1,
    vertexColors: true,
    transparent: true,
    opacity: 0.8
  })

  points = new THREE.Points(pointGeometry, pointMaterial)
  scene.add(points)

  // 创建连接线
  const linePositions = []
  const lineColors = []

  for (let i = 0; i < pointCount; i++) {
    for (let j = i + 1; j < pointCount; j++) {
      const distance = Math.sqrt(
        Math.pow(positions[i * 3] - positions[j * 3], 2) +
        Math.pow(positions[i * 3 + 1] - positions[j * 3 + 1], 2) +
        Math.pow(positions[i * 3 + 2] - positions[j * 3 + 2], 2)
      )

      if (distance < 3) {
        linePositions.push(
          positions[i * 3], positions[i * 3 + 1], positions[i * 3 + 2],
          positions[j * 3], positions[j * 3 + 1], positions[j * 3 + 2]
        )

        lineColors.push(
          colors[i * 3], colors[i * 3 + 1], colors[i * 3 + 2],
          colors[j * 3], colors[j * 3 + 1], colors[j * 3 + 2]
        )
      }
    }
  }

  const lineGeometry = new THREE.BufferGeometry()
  lineGeometry.setAttribute('position', new THREE.Float32BufferAttribute(linePositions, 3))
  lineGeometry.setAttribute('color', new THREE.Float32BufferAttribute(lineColors, 3))

  const lineMaterial = new THREE.LineBasicMaterial({
    vertexColors: true,
    transparent: true,
    opacity: 0.3
  })

  lines = new THREE.LineSegments(lineGeometry, lineMaterial)
  scene.add(lines)
}

const animate = () => {
  animationId = requestAnimationFrame(animate)

  time += 0.01

  if (points) {
    points.rotation.x += 0.001
    points.rotation.y += 0.002

    // 让点有轻微的浮动效果
    const positions = points.geometry.getAttribute('position')
    for (let i = 0; i < positions.count; i++) {
      const x = positions.getX(i)
      const y = positions.getY(i)
      const z = positions.getZ(i)

      positions.setX(i, x + Math.sin(time + i) * 0.01)
      positions.setY(i, y + Math.cos(time + i) * 0.01)
      positions.setZ(i, z + Math.sin(time + i * 0.5) * 0.01)
    }
    positions.needsUpdate = true
  }

  if (lines) {
    lines.rotation.x += 0.001
    lines.rotation.y += 0.002
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
.pointcloud-container {
  width: 100vw;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
  z-index: -1;
}
</style>