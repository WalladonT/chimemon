<script setup>
defineProps({
  monster: {
    type: Object,
    required: true,
  },
})

const statNames = [
  'HP',
  'ATK',
  'DEF',
  'FireATK',
  'WaterATK',
  'GrassATK',
  'FireDEF',
  'WaterDEF',
  'GrassDEF',
]

const moveNames = ['Move1', 'Move2', 'Move3', 'Move4']
</script>

<template>
  <article class="monster-card" :class="{ fainted: monster.currentHP <= 0 }">
    <header class="monster-header">
      <h3>{{ monster.Name }}</h3>
      <span>{{ monster.currentHP }} / {{ monster.HP }}</span>
    </header>

    <div class="hp-track" aria-label="Hit points">
      <span class="hp-fill" :style="{ width: `${monster.hpPercent}%` }"></span>
    </div>

    <dl class="stat-grid">
      <template v-for="statName in statNames" :key="statName">
        <dt>{{ statName }}</dt>
        <dd>{{ monster[statName] }}</dd>
      </template>
    </dl>

    <ul class="move-list" aria-label="Moves">
      <li v-for="moveName in moveNames" :key="moveName">
        <span>
          {{ monster[moveName].Name }}
          <small>{{ monster[moveName].Type }} {{ monster[moveName].Power }}</small>
        </span>
        <strong>{{ monster.cooldowns[moveName] }}s</strong>
      </li>
    </ul>
  </article>
</template>

<style scoped>
.monster-card {
  display: grid;
  gap: 10px;
  min-width: 0;
  padding: 12px;
  border: 1px solid rgba(255, 246, 213, 0.22);
  border-radius: 8px;
  background: rgba(9, 24, 28, 0.46);
  text-align: left;
}

.monster-card.fainted {
  opacity: 0.46;
}

.monster-header {
  display: flex;
  align-items: start;
  justify-content: space-between;
  gap: 10px;
}

.monster-header h3,
.monster-header span {
  margin: 0;
  font-size: 0.9rem;
}

.monster-header span {
  color: #f2c45f;
  font-weight: 900;
  white-space: nowrap;
}

.hp-track {
  height: 10px;
  overflow: hidden;
  border-radius: 999px;
  background: rgba(255, 246, 213, 0.16);
}

.hp-fill {
  display: block;
  height: 100%;
  border-radius: inherit;
  background: linear-gradient(90deg, #65d18d, #f2c45f);
  transition: width 180ms ease;
}

.stat-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 6px 8px;
  margin: 0;
}

.stat-grid dt {
  color: rgba(255, 246, 213, 0.68);
  font-size: 0.66rem;
  font-weight: 800;
}

.stat-grid dd {
  margin: 0;
  color: #fff6d5;
  font-size: 0.82rem;
  font-weight: 900;
}

.move-list {
  display: grid;
  gap: 6px;
  margin: 0;
  padding: 0;
  list-style: none;
}

.move-list li {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
  min-height: 34px;
  padding: 6px 8px;
  border-radius: 6px;
  background: rgba(255, 246, 213, 0.08);
}

.move-list span {
  display: grid;
  gap: 2px;
  min-width: 0;
  font-size: 0.78rem;
  font-weight: 900;
}

.move-list small {
  color: rgba(255, 246, 213, 0.7);
  font-size: 0.66rem;
  font-weight: 800;
}

.move-list strong {
  color: #9be7d8;
  font-size: 0.74rem;
}
</style>
