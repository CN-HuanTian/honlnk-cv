<script setup lang="ts">
  import type { ResumeData } from '@/types/types'
  import { ref } from 'vue'

  defineProps<{
    projects: ResumeData['projects']
  }>()

  const expandedProjects = ref<Set<string>>(new Set())
  const expandingHeight = ref<{ [key: string]: number }>({})
  const wordDict = ref<Record<string, boolean>>({})
  const titleMap = ref<Record<string, boolean>>({})
  const timerMap = ref<Record<string, number>>({})

  const toggleDetails = (title: string, event: MouseEvent, highlights: string[]) => {
    const card = (event.currentTarget as HTMLElement).closest('.project-card') as HTMLElement
    const details = card?.querySelector('.drawer-content') as HTMLElement

    if (expandedProjects.value.has(title)) {
      // 收起时保存当前高度
      if (details) {
        expandingHeight.value[title] = details.scrollHeight
      }
      expandedProjects.value.delete(title)
      stopAnimation(title, highlights)
      setTimeout(() => {
        wordDict.value={}
        timerMap.value={}
        titleMap.value={}
      }, 1000);
    } else {
      // 展开时计算并设置高度
      if (details) {
        expandingHeight.value[title] = details.scrollHeight
      }
      expandedProjects.value.add(title)
      animateHighlights(title, highlights)
    }
  }

  const getDrawerHeight = (title: string) => {
    if (expandedProjects.value.has(title)) {
      return expandingHeight.value[title] ? `${expandingHeight.value[title]}px` : 'auto'
    }
    return '0px'
  }

  async function animateHighlights(title: string, highlights: string[]) {
    for (let rowIndex = 0; rowIndex < highlights.length; rowIndex++) {
      const row = highlights[rowIndex]
      const chars = row.split('')
      // 为当前行创建所有 Promise
      const promises = chars.map((word, wordIndex) => {
        return new Promise(resolve => {
          const delay = wordIndex * 10
          const key = `${title}:${rowIndex}:${wordIndex}`

          const timer = setTimeout(() => {
            wordDict.value[key] = true
            resolve() // 标记完成
          }, delay)
          timerMap.value[`${title}:${rowIndex}:${wordIndex}`] = timer
        })
      })
      titleMap.value[row + rowIndex] = true
      await Promise.all(promises)
    }
  }

  function stopAnimation(title: string, highlights: string[]) {
    highlights.forEach((row, rowIndex) => {
      row.split('').forEach((_, wordIndex) => {
        const key = `${title}:${rowIndex}:${wordIndex}`
        if (timerMap.value[key] != undefined) {
          clearTimeout(timerMap.value[key])
          delete timerMap.value[key]
        }
      })
    })
  }
</script>

<template>
  <section class="section projects">
    <h2 class="section-title">🚀 项目经历</h2>

    <div
      v-for="(project, index) in projects"
      :key="project.title"
      class="project-card h-[500px]"
      :class="{
        'mb-4': index !== projects.length - 1,
        'before:opacity-0 after:opacity-100': expandedProjects.has(project.title),
      }"
      v-motion
      :initial="{ opacity: 0, x: -30 }"
      :visible-once="{ opacity: 1, x: 0 }"
      :transition="{
        delay: index * 200,
        duration: 600,
        type: 'spring',
        stiffness: 80,
      }"
      @click="toggleDetails(project.title, $event, project.highlights)"
    >
      <div class="card-header flex justify-between items-center p-6 cursor-pointer">
        <div class="header-left flex-1">
          <h3 class="text-xl font-semibold text-primary m-0">{{ project.title }}</h3>
          <span class="duration text-text-secondary text-sm">{{ project.duration }}</span>
        </div>
        <span
          class="role-tag bg-[rgb(var(--color-secondary))] text-[rgb(var(--color-gray-50))] px-3 py-1 rounded-full text-sm"
        >
          {{ project.role }}
        </span>
      </div>

      <!-- 抽屉容器 -->
      <div
        class="drawer-wrapper overflow-hidden transition-all duration-500 ease-in-out"
        :style="{
          height: getDrawerHeight(project.title),
          opacity: expandedProjects.has(project.title) ? 1 : 0,
        }"
      >
        <div class="drawer-content px-6 pb-6 border-t border-b-[rgb(var(--card-border))]">
          <!-- 项目亮点 -->
          <ul class="highlights my-4">
            <li
              v-for="(item, index) in project.highlights"
              :key="index"
              class="text-text-primary flex items-center"
            >
              <span v-if="titleMap[item + index]" class="text-secondary mr-2 mt-1">•</span>
              <div class="break-all flex w-full overflow-hidden flex-wrap">
                <template v-for="(v, k) in item.split('')" :key="k">
                  <!-- <span
                    :style="{
                      display: expandedProjects.has(project.title) ? 'none' : '0',
                      animation: expandedProjects.has(project.title)
                        ? `fadeIn 0.00001s ease-out ${k * 0.04}s forwards`
                        : `fadeOut ease-out ${(item.length - 1 - k) * 0.04}s forwards`,
                    }"
                  > -->
                  <span v-if="wordDict[`${project.title}:${index}:${k}`]">
                    {{ v }}
                  </span></template
                >
              </div>
            </li>
          </ul>
          <div class="tech-stack flex flex-wrap gap-2">
            <span
              v-for="(tech, index) in project.techStack"
              :key="`${tech}-${expandedProjects.has(project.title)}`"
              class="tech-tag"
              v-motion
              :initial="{ opacity: 0, x: -30 }"
              :enter="{
                opacity: 1,
                x: 0,
                transition: {
                  delay: 200 + index * 100,
                  duration: 600,
                  type: 'spring',
                  stiffness: 100,
                },
              }"
            >
              {{ tech }}
            </span>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>
<style>
  @keyframes fadeIn {
    to {
      display: block;
      opacity: 1;
    }
  }
  @keyframes fadeOut {
    to {
      opacity: 0;
    }
  }
</style>
