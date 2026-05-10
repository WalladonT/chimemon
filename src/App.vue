<script setup>
import { computed, onBeforeUnmount, ref } from 'vue'
import BattleScene from './components/scenes/BattleScene.vue'
import DestinationScene from './components/scenes/DestinationScene.vue'
import FindingEnemyScene from './components/scenes/FindingEnemyScene.vue'
import HomeScene from './components/scenes/HomeScene.vue'
import PrepareScene from './components/scenes/PrepareScene.vue'

const currentScene = ref('home')
const selectedDestination = ref('')
let enemySearchTimer = null

const destinationTitle = computed(() => selectedDestination.value)

function clearEnemySearchTimer() {
  if (enemySearchTimer) {
    clearTimeout(enemySearchTimer)
    enemySearchTimer = null
  }
}

function goToPrepare() {
  clearEnemySearchTimer()
  currentScene.value = 'prepare'
  selectedDestination.value = ''
}

function goToDestination(destination) {
  clearEnemySearchTimer()

  if (destination.id === 'battle') {
    currentScene.value = 'finding-enemy'
    selectedDestination.value = 'Battle'
    enemySearchTimer = setTimeout(() => {
      currentScene.value = 'battle'
      enemySearchTimer = null
    }, 5000)
    return
  }

  currentScene.value = 'destination'
  selectedDestination.value = destination.label
}

onBeforeUnmount(clearEnemySearchTimer)
</script>

<template>
  <main class="app-shell">
    <HomeScene v-if="currentScene === 'home'" @play="goToPrepare" />
    <PrepareScene v-else-if="currentScene === 'prepare'" @select-destination="goToDestination" />
    <FindingEnemyScene v-else-if="currentScene === 'finding-enemy'" @cancel="goToPrepare" />
    <BattleScene v-else-if="currentScene === 'battle'" @back="goToPrepare" />
    <DestinationScene v-else :title="destinationTitle" @back="goToPrepare" />
  </main>
</template>

<style>
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  min-width: 320px;
  min-height: 100vh;
  font-family:
    Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  color: #f8f4e8;
  background:
    radial-gradient(circle at 50% 18%, rgba(255, 221, 112, 0.16), transparent 28rem),
    linear-gradient(140deg, #1f413a 0%, #17252f 52%, #3b2433 100%);
}

button {
  font: inherit;
}

.app-shell {
  min-height: 100vh;
}

.scene {
  display: grid;
  min-height: 100vh;
  padding: 32px;
  place-items: center;
  text-align: center;
}

.scene-header {
  display: grid;
  gap: 8px;
}

.eyebrow {
  margin: 0;
  color: #f2c45f;
  font-size: 0.82rem;
  font-weight: 800;
  letter-spacing: 0.16em;
  text-transform: uppercase;
}

h1 {
  margin: 0;
  font-size: clamp(2.5rem, 8vw, 5.75rem);
  line-height: 0.95;
}

.primary-button,
.back-button {
  min-height: 56px;
  border: 0;
  border-radius: 8px;
  color: #17252f;
  background: #fff6d5;
  box-shadow: 0 14px 28px rgba(0, 0, 0, 0.22);
  cursor: pointer;
  font-weight: 900;
  transition:
    transform 150ms ease,
    background 150ms ease;
}

.primary-button:hover,
.back-button:hover {
  transform: translateY(-2px);
  background: #f2c45f;
}

.primary-button:focus-visible,
.back-button:focus-visible {
  outline: 4px solid #9be7d8;
  outline-offset: 6px;
}

.back-button {
  padding: 0 28px;
}

@media (max-width: 720px) {
  .scene {
    padding: 24px 16px;
  }
}
</style>
