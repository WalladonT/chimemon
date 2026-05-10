<script setup>
defineEmits(['play'])
</script>

<template>
  <section class="scene home-scene" aria-labelledby="home-title">
    <div class="title-block">
      <p class="eyebrow">Idle Adventure</p>
      <h1 id="home-title">Idle Chimemon</h1>
    </div>

    <button class="play-button" type="button" aria-label="Play" @click="$emit('play')">
      <span class="play-icon" aria-hidden="true"></span>
    </button>
  </section>
</template>

<style scoped>
.home-scene {
  grid-template-rows: 1fr auto 1fr;
  gap: 36px;
}

.title-block {
  display: grid;
  gap: 8px;
}

.play-button {
  position: relative;
  display: grid;
  width: min(46vw, 260px);
  aspect-ratio: 1;
  place-items: center;
  border: 8px solid #fff6d5;
  border-radius: 999px;
  color: #263535;
  background: linear-gradient(145deg, #ffe28a, #ef7d4c);
  box-shadow:
    0 28px 70px rgba(0, 0, 0, 0.38),
    inset 0 -14px 20px rgba(95, 40, 35, 0.2),
    inset 0 12px 18px rgba(255, 255, 255, 0.3);
  cursor: pointer;
  transition:
    transform 160ms ease,
    box-shadow 160ms ease;
}

.play-button:hover,
.play-button:focus-visible {
  transform: translateY(-4px) scale(1.03);
  box-shadow:
    0 34px 80px rgba(0, 0, 0, 0.46),
    inset 0 -14px 20px rgba(95, 40, 35, 0.2),
    inset 0 12px 18px rgba(255, 255, 255, 0.3);
}

.play-button:focus-visible {
  outline: 4px solid #9be7d8;
  outline-offset: 6px;
}

.play-icon {
  width: 0;
  height: 0;
  margin-left: 7%;
  border-top: min(8vw, 44px) solid transparent;
  border-bottom: min(8vw, 44px) solid transparent;
  border-left: min(13vw, 76px) solid currentColor;
  filter: drop-shadow(0 5px 0 rgba(255, 255, 255, 0.24));
}
</style>
