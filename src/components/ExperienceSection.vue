<template>
  <section id="experience" class="experience-section">
    <h2 class="section-title">项目经验</h2>
    <div class="timeline-container">
      <div class="timeline-line"></div>

      <div
        v-for="(project, index) in projects"
        :key="project.id"
        class="timeline-item"
        :class="{ 'left': index % 2 === 0, 'right': index % 2 === 1 }"
      >
        <div class="timeline-dot"></div>
        <div class="project-card">
          <div class="project-header">
            <h3 class="project-title">{{ project.title }}</h3>
            <span class="project-date">{{ project.date }}</span>
          </div>
          <p class="project-description">{{ project.description }}</p>
          <div class="project-tech">
            <span
              v-for="tech in project.technologies"
              :key="tech"
              class="tech-tag"
            >
              {{ tech }}
            </span>
          </div>
          <div class="project-links">
            <a
              v-if="project.demo"
              :href="project.demo"
              target="_blank"
              class="project-link demo-link"
            >
              <span class="link-icon">🚀</span>
              查看Demo
            </a>
            <a
              v-if="project.github"
              :href="project.github"
              target="_blank"
              class="project-link github-link"
            >
              <span class="link-icon">📁</span>
              GitHub
            </a>
          </div>
          <div class="project-achievements" v-if="project.achievements">
            <h4>主要成就：</h4>
            <ul>
              <li v-for="achievement in project.achievements" :key="achievement">
                {{ achievement }}
              </li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const projects = ref([
  {
    id: 1,
    title: '3D产品展示平台',
    date: '2024.01 - 2024.03',
    description: '基于Three.js开发的交互式3D产品展示平台，支持产品360度旋转、缩放、材质切换等功能。实现了高性能的3D渲染和流畅的用户交互体验。',
    technologies: ['Three.js', 'Vue.js', 'WebGL', 'TypeScript'],
    demo: 'https://demo.example.com',
    github: 'https://github.com/example/3d-showcase',
    achievements: [
      '实现了产品模型的实时渲染和交互',
      '优化了3D场景的加载性能，提升50%',
      '支持多种产品材质和光照效果'
    ]
  },
  {
    id: 2,
    title: '数据可视化仪表板',
    date: '2023.09 - 2023.12',
    description: '企业级数据可视化解决方案，集成多种图表类型，支持实时数据更新和交互式数据探索。为决策者提供直观的数据洞察。',
    technologies: ['D3.js', 'React', 'TypeScript', 'Node.js'],
    demo: 'https://demo.example.com/dashboard',
    github: 'https://github.com/example/data-viz',
    achievements: [
      '开发了15+种自定义图表组件',
      '实现了实时数据流处理',
      '支持大数据量的高效渲染'
    ]
  },
  {
    id: 3,
    title: 'Web游戏引擎',
    date: '2023.06 - 2023.08',
    description: '轻量级Web游戏引擎，提供完整的游戏开发框架，包括物理引擎、音频系统、资源管理等核心功能。',
    technologies: ['Three.js', 'Cannon.js', 'WebGL', 'Web Audio API'],
    demo: 'https://demo.example.com/game-engine',
    github: 'https://github.com/example/game-engine',
    achievements: [
      '实现了完整的物理碰撞检测系统',
      '开发了音频管理和3D音效功能',
      '支持多平台游戏发布'
    ]
  },
  {
    id: 4,
    title: '地理信息系统',
    date: '2023.03 - 2023.05',
    description: '基于CesiumJS的3D地理信息系统，支持地图可视化、空间分析、路径规划等功能。为城市规划提供技术支持。',
    technologies: ['CesiumJS', 'Vue.js', 'PostgreSQL', 'PostGIS'],
    demo: 'https://demo.example.com/gis',
    github: 'https://github.com/example/gis-system',
    achievements: [
      '集成了多种地理数据源',
      '实现了3D地形和建筑模型展示',
      '开发了空间查询和分析功能'
    ]
  },
  {
    id: 5,
    title: '实时协作平台',
    date: '2022.11 - 2023.02',
    description: '支持多用户实时协作的在线平台，包含文档编辑、项目管理、团队沟通等功能。采用WebSocket实现实时同步。',
    technologies: ['Vue.js', 'Node.js', 'Socket.io', 'MongoDB'],
    demo: 'https://demo.example.com/collaboration',
    github: 'https://github.com/example/collab-platform',
    achievements: [
      '实现了实时文档协作编辑',
      '支持100+用户同时在线',
      '开发了完整的权限管理系统'
    ]
  },
  {
    id: 6,
    title: 'AI图像处理应用',
    date: '2022.08 - 2022.10',
    description: '基于机器学习的图像处理应用，支持图像识别、风格转换、智能裁剪等功能。前端采用Canvas API进行图像处理。',
    technologies: ['React', 'TensorFlow.js', 'Canvas API', 'Python'],
    demo: 'https://demo.example.com/ai-image',
    github: 'https://github.com/example/ai-image-app',
    achievements: [
      '集成了多种AI模型进行图像处理',
      '实现了客户端图像预处理优化',
      '支持批量图像处理功能'
    ]
  }
])
</script>

<style scoped>
.experience-section {
  width: 100vw;
  min-height: 100vh;
  background: #111;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 4rem 0;
}

.section-title {
  font-size: 2.5rem;
  margin-bottom: 3rem;
  color: #667eea;
  text-align: center;
  font-weight: 700;
}

.timeline-container {
  position: relative;
  width: 100%;
  max-width: 1200px;
  padding: 0 2rem;
}

.timeline-line {
  position: absolute;
  left: 50%;
  top: 0;
  bottom: 0;
  width: 4px;
  background: linear-gradient(180deg, #667eea, #42b883, #ff6600);
  transform: translateX(-50%);
  border-radius: 2px;
}

.timeline-item {
  position: relative;
  margin-bottom: 4rem;
  width: 45%;
}

.timeline-item.left {
  left: 0;
  text-align: right;
}

.timeline-item.right {
  left: 55%;
  text-align: left;
}

.timeline-dot {
  position: absolute;
  top: 20px;
  width: 20px;
  height: 20px;
  background: #667eea;
  border: 4px solid #111;
  border-radius: 50%;
  z-index: 2;
}

.timeline-item.left .timeline-dot {
  right: -52px;
}

.timeline-item.right .timeline-dot {
  left: -52px;
}

.project-card {
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 15px;
  padding: 2rem;
  backdrop-filter: blur(10px);
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.project-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 3px;
  background: linear-gradient(90deg, #667eea, #42b883);
  opacity: 0;
  transition: opacity 0.3s ease;
}

.project-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
  border-color: rgba(102, 126, 234, 0.3);
}

.project-card:hover::before {
  opacity: 1;
}

.project-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 1rem;
  flex-wrap: wrap;
  gap: 1rem;
}

.project-title {
  color: #fff;
  font-size: 1.4rem;
  font-weight: 700;
  margin: 0;
  line-height: 1.3;
}

.project-date {
  color: #667eea;
  font-size: 0.9rem;
  font-weight: 600;
  background: rgba(102, 126, 234, 0.1);
  padding: 0.3rem 0.8rem;
  border-radius: 20px;
  border: 1px solid rgba(102, 126, 234, 0.3);
}

.project-description {
  color: #ccc;
  line-height: 1.6;
  margin-bottom: 1.5rem;
  font-size: 0.95rem;
}

.project-tech {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin-bottom: 1.5rem;
}

.tech-tag {
  background: rgba(102, 126, 234, 0.2);
  color: #667eea;
  padding: 0.3rem 0.8rem;
  border-radius: 15px;
  font-size: 0.8rem;
  font-weight: 600;
  border: 1px solid rgba(102, 126, 234, 0.3);
  transition: all 0.3s ease;
}

.tech-tag:hover {
  background: rgba(102, 126, 234, 0.3);
  transform: translateY(-1px);
}

.project-links {
  display: flex;
  gap: 1rem;
  margin-bottom: 1.5rem;
  flex-wrap: wrap;
}

.project-link {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.6rem 1.2rem;
  border-radius: 8px;
  text-decoration: none;
  font-weight: 600;
  font-size: 0.9rem;
  transition: all 0.3s ease;
  border: 1px solid transparent;
}

.demo-link {
  background: rgba(66, 184, 131, 0.2);
  color: #42b883;
  border-color: rgba(66, 184, 131, 0.3);
}

.demo-link:hover {
  background: rgba(66, 184, 131, 0.3);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(66, 184, 131, 0.2);
}

.github-link {
  background: rgba(255, 255, 255, 0.1);
  color: #fff;
  border-color: rgba(255, 255, 255, 0.2);
}

.github-link:hover {
  background: rgba(255, 255, 255, 0.2);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(255, 255, 255, 0.1);
}

.link-icon {
  font-size: 1rem;
}

.project-achievements {
  border-top: 1px solid rgba(255, 255, 255, 0.1);
  padding-top: 1rem;
}

.project-achievements h4 {
  color: #fff;
  font-size: 1rem;
  margin-bottom: 0.8rem;
  font-weight: 600;
}

.project-achievements ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

.project-achievements li {
  color: #ccc;
  font-size: 0.9rem;
  line-height: 1.5;
  margin-bottom: 0.5rem;
  padding-left: 1.2rem;
  position: relative;
}

.project-achievements li::before {
  content: '✓';
  position: absolute;
  left: 0;
  color: #42b883;
  font-weight: bold;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .timeline-line {
    left: 20px;
  }

  .timeline-item {
    width: calc(100% - 60px);
    left: 60px !important;
    text-align: left !important;
  }

  .timeline-item.left .timeline-dot,
  .timeline-item.right .timeline-dot {
    left: -40px;
    right: auto;
  }

  .project-header {
    flex-direction: column;
    align-items: flex-start;
  }

  .project-links {
    flex-direction: column;
  }

  .project-link {
    justify-content: center;
  }
}

@media (max-width: 480px) {
  .section-title {
    font-size: 2rem;
  }

  .project-card {
    padding: 1.5rem;
  }

  .project-title {
    font-size: 1.2rem;
  }
}
</style>