<script setup lang="ts">
import HeroSection from '@/components/HeroSection.vue'
import ProjectsSection from '@/components/ProjectsSection.vue'
import SkillsSection from '@/components/SkillsSection.vue'
import ExperienceSection from '@/components/ExperienceSection.vue'
import FooterLogo from '@/components/FooterLogo.vue'
</script>

<template>
  <main>
    <HeroSection />
    <ProjectsSection />
    <SkillsSection />
    <ExperienceSection />
    <FooterLogo />
  </main>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: #000;
  color: white;
  overflow-x: hidden;
}

main {
  width: 100vw;
  min-height: 100vh;
}
</style>
