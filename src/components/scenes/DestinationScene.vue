<script setup>
defineProps({
  title: {
    type: String,
    required: true,
  },
})

defineEmits(['back'])
</script>

<template>
  <section class="scene destination-scene" :aria-labelledby="`${title.toLowerCase()}-title`">
    <header class="scene-header">
      <p class="eyebrow">Destination</p>
      <h1 :id="`${title.toLowerCase()}-title`">{{ title }}</h1>
    </header>

    <div class="destination-panel">
      <p>{{ title }} scene placeholder</p>
    </div>

    <button class="back-button" type="button" @click="$emit('back')">Back to Prepare</button>
  </section>
</template>

<style scoped>
.destination-scene {
  align-content: center;
  gap: 28px;
}

.destination-panel {
  display: grid;
  width: min(86vw, 420px);
  min-height: 220px;
  place-items: center;
  padding: 28px;
  border-radius: 8px;
  background: rgba(9, 24, 28, 0.38);
}

.destination-panel p {
  margin: 0;
  color: #fff6d5;
  font-weight: 800;
}
</style>
