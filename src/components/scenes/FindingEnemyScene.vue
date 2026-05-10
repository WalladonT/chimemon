<script setup>
defineEmits(['cancel'])
</script>

<template>
  <section class="scene finding-scene" aria-labelledby="finding-title">
    <header class="scene-header">
      <p class="eyebrow">Battle</p>
      <h1 id="finding-title">Finding Enemy</h1>
    </header>

    <div class="search-panel" role="status" aria-live="polite">
      <span class="spinner" aria-hidden="true"></span>
      <p>Searching for an enemy...</p>
    </div>

    <button class="back-button" type="button" @click="$emit('cancel')">Cancel</button>
  </section>
</template>

<style scoped>
.finding-scene {
  align-content: center;
  gap: 28px;
}

.search-panel {
  display: grid;
  width: min(86vw, 420px);
  min-height: 220px;
  place-items: center;
  gap: 20px;
  padding: 28px;
  border-radius: 8px;
  background: rgba(9, 24, 28, 0.38);
}

.spinner {
  width: 86px;
  height: 86px;
  border: 10px solid rgba(255, 246, 213, 0.22);
  border-top-color: #f2c45f;
  border-radius: 999px;
  animation: spin 1s linear infinite;
}

.search-panel p {
  margin: 0;
  color: #fff6d5;
  font-weight: 900;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
</style>
