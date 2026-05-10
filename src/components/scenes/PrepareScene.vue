<script setup>
defineEmits(['select-destination'])

const destinations = [
  { id: 'shop', label: 'Shop' },
  { id: 'blacksmith', label: 'Blacksmith' },
  { id: 'training', label: 'Training' },
  { id: 'battle', label: 'Battle' },
]
</script>

<template>
  <section class="scene prepare-scene" aria-labelledby="prepare-title">
    <header class="scene-header">
      <p class="eyebrow">Ready Room</p>
      <h1 id="prepare-title">Prepare Scene</h1>
    </header>

    <div class="monster-placeholder" aria-label="Monster placeholder">
      <div class="monster-shadow"></div>
      <div class="monster-body">
        <span class="monster-eye"></span>
        <span class="monster-eye"></span>
      </div>
      <p>Monster Placeholder</p>
    </div>

    <nav class="destination-grid" aria-label="Prepare destinations">
      <button
        v-for="destination in destinations"
        :key="destination.id"
        class="primary-button"
        type="button"
        @click="$emit('select-destination', destination)"
      >
        {{ destination.label }}
      </button>
    </nav>
  </section>
</template>

<style scoped>
.prepare-scene {
  align-content: center;
  gap: 28px;
}

.monster-placeholder {
  display: grid;
  width: min(86vw, 420px);
  min-height: 310px;
  place-items: center;
  gap: 16px;
  padding: 28px;
  border: 3px dashed rgba(255, 246, 213, 0.48);
  border-radius: 8px;
  background: rgba(9, 24, 28, 0.38);
  box-shadow: inset 0 0 32px rgba(255, 255, 255, 0.04);
}

.monster-body {
  display: flex;
  width: 160px;
  height: 140px;
  align-items: center;
  justify-content: center;
  gap: 32px;
  border: 4px solid #fff6d5;
  border-radius: 46% 54% 42% 58%;
  background: linear-gradient(155deg, #80d2a9, #40857a);
  box-shadow: inset 0 -16px 0 rgba(0, 0, 0, 0.12);
}

.monster-eye {
  width: 22px;
  height: 22px;
  border-radius: 999px;
  background: #1f2426;
}

.monster-shadow {
  width: 180px;
  height: 20px;
  align-self: end;
  border-radius: 999px;
  background: rgba(0, 0, 0, 0.25);
  filter: blur(2px);
}

.monster-placeholder p {
  margin: 0;
  color: #fff6d5;
  font-weight: 800;
}

.destination-grid {
  display: grid;
  width: min(90vw, 620px);
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 12px;
}

@media (max-width: 720px) {
  .destination-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}

@media (max-width: 420px) {
  .destination-grid {
    grid-template-columns: 1fr;
  }
}
</style>
