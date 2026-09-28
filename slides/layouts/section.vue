<script setup lang="ts">
defineProps<{
  round?: string
}>()
</script>

<template>
  <div class="slidev-layout bz-section">
    <div class="bz-round-rule"><span>{{ round }}</span></div>
    <div class="bz-section-body">
      <slot />
    </div>
  </div>
</template>

<style scoped>
.bz-section {
  display: flex;
  flex-direction: column;
  justify-content: center;
  gap: 2.5rem;
  background: var(--bz-bg);
}

.bz-section-body {
  text-align: center;
}

.bz-section-body :deep(h1) {
  font-size: 3rem;
}

.bz-section-body :deep(h1)::after {
  margin-left: auto;
  margin-right: auto;
}

.bz-section-body :deep(p) {
  color: var(--bz-muted);
  font-size: 1.25rem;
}
</style>
