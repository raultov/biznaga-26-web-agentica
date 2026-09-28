<script setup lang="ts">
defineProps<{
  eyebrow?: string
  speaker?: string
  role?: string
  when?: string
}>()
</script>

<template>
  <div class="slidev-layout bz-cover">
    <img class="bz-cover-logo" src="/images/biznaga-logo.svg" alt="BiznagaFest">
    <div class="bz-cover-body">
      <div v-if="eyebrow" class="bz-eyebrow">{{ eyebrow }}</div>
      <slot />
    </div>
    <div class="bz-cover-footer">
      <div>
        <div class="bz-cover-speaker">{{ speaker }}</div>
        <div class="bz-cover-role">{{ role }}</div>
      </div>
      <div class="bz-eyebrow">{{ when }}</div>
    </div>
  </div>
</template>

<style scoped>
.bz-cover {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  background:
    radial-gradient(ellipse at 85% 110%, rgba(222, 18, 35, 0.35), transparent 55%),
    radial-gradient(ellipse at 10% -10%, rgba(3, 3, 138, 0.9), transparent 60%),
    var(--bz-bg);
}

.bz-cover-logo {
  height: 2.2rem;
  align-self: flex-start;
}

.bz-cover-body :deep(h1) {
  font-size: 3.1rem;
  margin: 0.8rem 0 0.6rem;
}

.bz-cover-body :deep(p) {
  font-size: 1.3rem;
  color: var(--bz-muted);
}

.bz-cover-footer {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  border-top: 3px solid transparent;
  border-image: var(--bz-gradient) 1;
  padding-top: 0.8rem;
}

.bz-cover-speaker {
  font-family: var(--bz-font-heading);
  font-weight: 700;
  font-size: 1.4rem;
}

.bz-cover-role {
  color: var(--bz-blue);
  font-size: 0.95rem;
}
</style>
