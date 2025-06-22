<template>
  <nav class="navigation">
    <div class="nav-container">
      <div class="nav-buttons">
        <button
          class="nav-btn"
          @click="scrollToSection('skills')"
          :class="{ active: activeSection === 'skills' }"
        >
          <span class="nav-icon">💻</span>
          技能展示
        </button>
        <button
          class="nav-btn"
          @click="scrollToSection('experience')"
          :class="{ active: activeSection === 'experience' }"
        >
          <span class="nav-icon">📄</span>
          个人简历
        </button>
      </div>
    </div>
  </nav>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const activeSection = ref('')

const scrollToSection = (sectionId: string) => {
  const element = document.getElementById(sectionId)
  if (element) {
    element.scrollIntoView({
      behavior: 'smooth',
      block: 'start'
    })
  }
}

const updateActiveSection = () => {
  const sections = ['skills', 'experience']
  const scrollPosition = window.scrollY + 100

  for (const sectionId of sections) {
    const element = document.getElementById(sectionId)
    if (element) {
      const rect = element.getBoundingClientRect()
      const elementTop = rect.top + window.scrollY
      const elementBottom = elementTop + rect.height

      if (scrollPosition >= elementTop && scrollPosition < elementBottom) {
        activeSection.value = sectionId
        break
      }
    }
  }
}

onMounted(() => {
  window.addEventListener('scroll', updateActiveSection)
  updateActiveSection() // 初始化
})

onUnmounted(() => {
  window.removeEventListener('scroll', updateActiveSection)
})
</script>

<style scoped>
.navigation {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 1000;
  background: rgba(0, 0, 0, 0.8);
  backdrop-filter: blur(10px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.nav-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 2rem;
}

.nav-buttons {
  display: flex;
  gap: 1rem;
  padding: 1rem 0;
  justify-content: center;
}

.nav-btn {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.8rem 1.5rem;
  background: rgba(255, 255, 255, 0.05);
  border: 2px solid rgba(255, 255, 255, 0.1);
  border-radius: 25px;
  color: #ccc;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  backdrop-filter: blur(10px);
}

.nav-btn:hover {
  background: rgba(102, 126, 234, 0.2);
  border-color: rgba(102, 126, 234, 0.5);
  color: #fff;
  transform: translateY(-2px);
  box-shadow: 0 4px 15px rgba(102, 126, 234, 0.3);
}

.nav-btn.active {
  background: linear-gradient(135deg, #667eea, #42b883);
  border-color: transparent;
  color: #fff;
  box-shadow: 0 4px 20px rgba(102, 126, 234, 0.4);
}

.nav-icon {
  font-size: 1.2rem;
}

@media (max-width: 768px) {
  .nav-buttons {
    gap: 0.5rem;
  }

  .nav-btn {
    padding: 0.6rem 1rem;
    font-size: 0.9rem;
  }

  .nav-icon {
    font-size: 1rem;
  }
}

@media (max-width: 480px) {
  .nav-buttons {
    flex-direction: column;
    align-items: center;
  }

  .nav-btn {
    width: 200px;
    justify-content: center;
  }
}
</style>