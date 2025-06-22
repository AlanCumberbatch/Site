<template>
  <div ref="containerRef" class="fluid-container"></div>
</template>

<script lang="ts" setup>
import { onMounted, ref, onUnmounted } from 'vue'
import * as THREE from 'three'

const containerRef = ref<HTMLElement | null>(null)
let scene: THREE.Scene
let camera: THREE.PerspectiveCamera
let renderer: THREE.WebGLRenderer
let mesh: THREE.Mesh
let animationId: number
let time = 0

// 顶点着色器
const vertexShader = `
  varying vec2 vUv;
  void main() {
    vUv = uv;
    gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
  }
`

// 片段着色器 - 流体效果
const fragmentShader = `
  uniform float time;
  varying vec2 vUv;

  void main() {
    vec2 uv = vUv;

    // 创建流体动画
    float noise = sin(uv.x * 10.0 + time) * sin(uv.y * 10.0 + time * 0.5);
    noise += sin(uv.x * 20.0 + time * 2.0) * sin(uv.y * 20.0 + time * 1.5) * 0.5;
    noise += sin(uv.x * 30.0 + time * 3.0) * sin(uv.y * 30.0 + time * 2.5) * 0.25;

    // 颜色渐变
    vec3 color1 = vec3(0.1, 0.3, 0.8);
    vec3 color2 = vec3(0.8, 0.1, 0.5);
    vec3 color3 = vec3(0.1, 0.8, 0.3);

    float t = (noise + 1.0) * 0.5;
    vec3 color = mix(color1, color2, t);
    color = mix(color, color3, sin(time * 0.5) * 0.5 + 0.5);

    gl_FragColor = vec4(color, 0.8);
  }
`

const initThree = () => {
  scene = new THREE.Scene()
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
  camera.position.z = 1

  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setClearColor(0x000000, 0.1)
  containerRef.value?.appendChild(renderer.domElement)

  createFluidMesh()
  animate()
}

const createFluidMesh = () => {
  const geometry = new THREE.PlaneGeometry(2, 2)

  const material = new THREE.ShaderMaterial({
    vertexShader,
    fragmentShader,
    uniforms: {
      time: { value: 0 }
    },
    transparent: true
  })

  mesh = new THREE.Mesh(geometry, material)
  scene.add(mesh)
}

const animate = () => {
  animationId = requestAnimationFrame(animate)

  time += 0.01

  if (mesh && mesh.material instanceof THREE.ShaderMaterial) {
    mesh.material.uniforms.time.value = time
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
.fluid-container {
  width: 100vw;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
  z-index: -1;
}
</style>