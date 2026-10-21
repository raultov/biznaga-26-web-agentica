<script setup lang="ts">
import { computed } from 'vue'
import { useNav } from '@slidev/client'

const { currentPage, total, currentLayout, currentFrontmatter, clicks } = useNav()

// Demo slides reveal a full-slide video on the first click: keep the footer off it.
const hidden = computed(() =>
  ['cover', 'section', 'end'].includes(currentLayout.value)
  || (currentFrontmatter.value?.demoVideo && clicks.value > 0),
)
const stage = computed(() => String(currentPage.value).padStart(2, '0'))
</script>

<template>
  <footer v-if="!hidden" class="bz-footer">
    <span>BIZNAGAFEST 2026 · WEB-MCP</span>
    <span>STAGE {{ stage }}/{{ total }}</span>
  </footer>
</template>
