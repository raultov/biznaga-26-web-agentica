<template>
  <div class="slidev-layout bz-end">
    <div class="bz-end-title bz-blink">CONTINUE?</div>
    <div class="bz-end-body">
      <slot />
    </div>
    <div class="bz-eyebrow">Insert coin to ask a question</div>
  </div>
</template>

<style scoped>
.bz-end {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 2rem;
  text-align: center;
  background:
    radial-gradient(ellipse at 50% 120%, rgba(222, 18, 35, 0.35), transparent 60%),
    var(--bz-bg);
}

.bz-end-title {
  font-family: var(--bz-font-display);
  font-size: 2.6rem;
  color: var(--bz-yellow);
  /* Pixel drop shadow instead of background-clip: exports cleanly to PDF/PPTX. */
  text-shadow: 4px 4px 0 var(--bz-red);
}

.bz-end-body :deep(h1)::after {
  margin-left: auto;
  margin-right: auto;
}
</style>
