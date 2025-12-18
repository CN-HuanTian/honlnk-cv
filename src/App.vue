<script setup lang="ts">
  import AdditionalValue from '@/components/AdditionalValue.vue'
  import CoreAdvantages from '@/components/CoreAdvantages.vue'
  import EducationBackground from '@/components/EducationBackground.vue'
  import GitHubButton from '@/components/GitHubButton.vue'
  import Header from '@/components/Header.vue'
  import ProjectExperience from '@/components/ProjectExperience.vue'
  import ThemeToggle from '@/components/ThemeToggle.vue'
  import { useResumeData } from '@/composables/useResumeData'
  import { onMounted } from 'vue'

  const { resumeData, isLoading, hasError, isEmpty, loadResumeData } = useResumeData()

  // 页面加载时获取简历数据
  onMounted(() => {
    loadResumeData()
  })
</script>

<template>
  <!-- 应用根容器 -->
  <div class="app-container">
    <!-- GitHub 按钮 -->
    <div class="github-button-wrapper">
      <GitHubButton show-text />
    </div>

    <!-- 主题切换按钮 -->
    <div class="theme-toggle-wrapper">
      <ThemeToggle />
    </div>

    <!-- 简历容器 -->
    <div class="resume-container">
      <!-- 加载状态 -->
      <div v-if="isLoading" class="loading-state">
        <div class="state-icon">⏳</div>
        <div class="loading-title">正在加载简历数据...</div>
      </div>

      <!-- 错误状态 -->
      <div v-else-if="hasError" class="error-state">
        <div class="state-icon">⚠️</div>
        <div class="error-title">加载失败</div>
        <div class="error-message">抱歉，无法加载简历数据。请检查网络连接或稍后重试。</div>
        <button @click="loadResumeData" class="retry-btn">
          <span>🔄</span>
          <span>重新加载</span>
        </button>
      </div>

      <!-- 空数据状态 -->
      <div v-else-if="isEmpty" class="empty-state">
        <div class="state-icon">📄</div>
        <div class="empty-title">暂无简历数据</div>
        <div class="empty-message">请检查 README.md 文件是否存在并包含有效的简历信息。</div>
      </div>

      <!-- 正常内容 -->
      <template v-else-if="resumeData">
        <!-- <Header :data="resumeData" /> -->

        <main class="content flex flex-col gap-8">
          <!-- <CoreAdvantages :advantages="resumeData.coreAdvantages" /> -->
          <ProjectExperience :projects="resumeData.projects" />
          <!-- <EducationBackground :data="resumeData.education" /> -->
          <!-- <AdditionalValue :values="resumeData.additionalValues" /> -->
        </main>
      </template>
    </div>
  </div>
</template>

<style>
  @import '@/styles/main.css';

  /* GitHub 按钮定位 */
  .github-button-wrapper {
    position: fixed;
    top: var(--spacing-lg);
    left: var(--spacing-lg);
    z-index: 1000;
    display: flex;
    align-items: center;
    justify-content: flex-start;
  }

  /* 主题切换按钮定位 */
  .theme-toggle-wrapper {
    position: fixed;
    top: var(--spacing-lg);
    right: var(--spacing-lg);
    z-index: 1000;
    display: flex;
    align-items: center;
    justify-content: flex-end;
  }

  /* 响应式调整 */
  @media (max-width: 768px) {
    .github-button-wrapper {
      top: var(--spacing-md);
      left: var(--spacing-md);
    }

    .theme-toggle-wrapper {
      top: var(--spacing-md);
      right: var(--spacing-md);
    }
  }

  @media (max-width: 480px) {
    .github-button-wrapper {
      top: var(--spacing-sm);
      left: var(--spacing-sm);
    }

    .theme-toggle-wrapper {
      top: var(--spacing-sm);
      right: var(--spacing-sm);
    }
  }

  /* 打印时隐藏装饰性按钮 */
  @media print {
    .github-button-wrapper {
      display: none;
    }

    .theme-toggle-wrapper {
      display: none;
    }
  }
</style>
