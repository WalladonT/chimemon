<script setup>
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'
import MonsterCard from '../battle/MonsterCard.vue'
import { enemyTeam, playerTeam } from '../../data/battleTeams'

defineEmits(['back'])

const moveNames = ['Move1', 'Move2', 'Move3', 'Move4']
const viewMode = ref('battle')
const playerMonsters = ref(createBattleTeam(playerTeam, 'player'))
const enemyMonsters = ref(createBattleTeam(enemyTeam, 'enemy'))
const battleLog = ref(['Battle started.'])
let battleTimer = null
let animationTimers = []

const playerAliveCount = computed(() => getAliveMonsters(playerMonsters.value).length)
const enemyAliveCount = computed(() => getAliveMonsters(enemyMonsters.value).length)
const activePlayerMonster = computed(() => getAliveMonsters(playerMonsters.value)[0] || playerMonsters.value[0])
const activeEnemyMonster = computed(() => getAliveMonsters(enemyMonsters.value)[0] || enemyMonsters.value[0])
const winner = computed(() => {
  if (playerAliveCount.value === 0) {
    return 'Enemy Team Wins'
  }

  if (enemyAliveCount.value === 0) {
    return 'Player Team Wins'
  }

  return ''
})

function createBattleTeam(team, side) {
  return team.map((monster) => ({
    ...structuredClone(monster),
    side,
    currentHP: monster.HP,
    cooldowns: {
      Move1: 0,
      Move2: 0,
      Move3: 0,
      Move4: 0,
    },
    hpPercent: 100,
    spriteState: '',
    damageIndicators: [],
    damageIndicatorId: 0,
  }))
}

function getAliveMonsters(team) {
  return team.filter((monster) => monster.currentHP > 0)
}

function getReadyMove(monster) {
  return moveNames.map((moveName) => ({ moveName, move: monster[moveName] })).find(({ moveName }) => {
    return monster.cooldowns[moveName] <= 0
  })
}

function getAttackStat(monster, moveType) {
  if (moveType === 'Physical') {
    return monster.ATK
  }

  return monster[`${moveType}ATK`]
}

function getDefenseStat(monster, moveType) {
  if (moveType === 'Physical') {
    return monster.DEF
  }

  return monster[`${moveType}DEF`]
}

function calculateDamage(attacker, defender, move) {
  const attack = getAttackStat(attacker, move.Type)
  const defense = getDefenseStat(defender, move.Type)

  return Math.max(1, Math.round(move.Power + attack - defense))
}

function pushBattleLog(message) {
  battleLog.value = [message, ...battleLog.value].slice(0, 40)
}

function queueAnimationTimer(callback, delay) {
  const timer = setTimeout(callback, delay)
  animationTimers.push(timer)
}

function clearMonsterAnimations() {
  animationTimers.forEach((timer) => clearTimeout(timer))
  animationTimers = []

  playerMonsters.value.forEach(clearMonsterAnimation)
  enemyMonsters.value.forEach(clearMonsterAnimation)
}

function clearMonsterAnimation(monster) {
  monster.spriteState = ''
  monster.damageIndicators = []
}

function triggerMonsterAnimation(attacker, target, damage) {
  const indicator = {
    id: `${target.Name}-${target.damageIndicatorId}`,
    value: damage,
  }

  target.damageIndicatorId += 1
  target.damageIndicators.push(indicator)
  attacker.spriteState = ''
  target.spriteState = ''

  queueAnimationTimer(() => {
    attacker.spriteState = 'attacking'
    target.spriteState = 'hit'
  }, 0)

  queueAnimationTimer(() => {
    attacker.spriteState = ''
    target.spriteState = ''
  }, 360)

  queueAnimationTimer(() => {
    target.damageIndicators = target.damageIndicators.filter((item) => item.id !== indicator.id)
  }, 760)
}

function tickCooldowns(team) {
  team.forEach((monster) => {
    if (monster.currentHP <= 0) {
      return
    }

    moveNames.forEach((moveName) => {
      monster.cooldowns[moveName] = Math.max(0, monster.cooldowns[moveName] - 1)
    })
  })
}

function attackWithTeam(attackingTeam, defendingTeam) {
  getAliveMonsters(attackingTeam).forEach((attacker) => {
    const target = getAliveMonsters(defendingTeam)[0]

    if (!target) {
      return
    }

    const readyMove = getReadyMove(attacker)

    if (!readyMove) {
      return
    }

    const damage = calculateDamage(attacker, target, readyMove.move)
    target.currentHP = Math.max(0, target.currentHP - damage)
    target.hpPercent = Math.max(0, Math.round((target.currentHP / target.HP) * 100))
    attacker.cooldowns[readyMove.moveName] = readyMove.move.Cooldown

    triggerMonsterAnimation(attacker, target, damage)
    pushBattleLog(`${attacker.Name} used ${readyMove.move.Name} on ${target.Name} for ${damage} damage.`)

    if (target.currentHP === 0) {
      pushBattleLog(`${target.Name} fainted.`)
    }
  })
}

function tickBattle() {
  if (winner.value) {
    stopBattle()
    return
  }

  tickCooldowns(playerMonsters.value)
  tickCooldowns(enemyMonsters.value)
  attackWithTeam(playerMonsters.value, enemyMonsters.value)
  attackWithTeam(enemyMonsters.value, playerMonsters.value)

  if (winner.value) {
    stopBattle()
  }
}

function startBattle() {
  stopBattle()
  battleTimer = setInterval(tickBattle, 1000)
}

function stopBattle() {
  if (battleTimer) {
    clearInterval(battleTimer)
    battleTimer = null
  }
}

function resetBattle() {
  clearMonsterAnimations()
  playerMonsters.value = createBattleTeam(playerTeam, 'player')
  enemyMonsters.value = createBattleTeam(enemyTeam, 'enemy')
  battleLog.value = ['Battle restarted.']
  viewMode.value = 'battle'
  startBattle()
}

onMounted(startBattle)
onBeforeUnmount(() => {
  stopBattle()
  clearMonsterAnimations()
})
</script>

<template>
  <section class="scene battle-scene" aria-labelledby="battle-title">
    <header class="battle-header">
      <div class="scene-header">
        <p class="eyebrow">Idle Battle</p>
        <h1 id="battle-title">Battle Scene</h1>
        <p class="battle-status">Player {{ playerAliveCount }}/6 vs Enemy {{ enemyAliveCount }}/6</p>
      </div>

      <div class="header-actions">
        <button class="primary-button" type="button" @click="viewMode = viewMode === 'battle' ? 'teams' : 'battle'">
          {{ viewMode === 'battle' ? 'Team Details' : 'Battle View' }}
        </button>
      </div>
    </header>

    <template v-if="viewMode === 'battle'">
      <div class="battle-view">
        <div class="sprite-arena" aria-label="Monster fight">
          <section class="sprite-team player-team-strip" aria-labelledby="player-sprites-title">
            <h2 id="player-sprites-title">Player Team</h2>
            <div class="sprite-row">
              <article
                v-for="monster in playerMonsters"
                :key="monster.Name"
                class="sprite-slot"
                :class="{ active: monster === activePlayerMonster, fainted: monster.currentHP <= 0 }"
              >
                <div class="sprite-name">{{ monster.Name }}</div>
                <div class="sprite player-sprite" :class="monster.spriteState">
                  <span class="sprite-eye"></span>
                  <span class="sprite-eye"></span>
                  <span
                    v-for="indicator in monster.damageIndicators"
                    :key="indicator.id"
                    class="damage-indicator"
                  >
                    -{{ indicator.value }}
                  </span>
                </div>
                <div class="sprite-hp">
                  <span :style="{ width: `${monster.hpPercent}%` }"></span>
                </div>
              </article>
            </div>
          </section>

          <div class="arena-center">
            <span>VS</span>
          </div>

          <section class="sprite-team enemy-team-strip" aria-labelledby="enemy-sprites-title">
            <h2 id="enemy-sprites-title">Enemy Team</h2>
            <div class="sprite-row">
              <article
                v-for="monster in enemyMonsters"
                :key="monster.Name"
                class="sprite-slot"
                :class="{ active: monster === activeEnemyMonster, fainted: monster.currentHP <= 0 }"
              >
                <div class="sprite-name">{{ monster.Name }}</div>
                <div class="sprite enemy-sprite" :class="monster.spriteState">
                  <span class="sprite-eye"></span>
                  <span class="sprite-eye"></span>
                  <span
                    v-for="indicator in monster.damageIndicators"
                    :key="indicator.id"
                    class="damage-indicator"
                  >
                    -{{ indicator.value }}
                  </span>
                </div>
                <div class="sprite-hp">
                  <span :style="{ width: `${monster.hpPercent}%` }"></span>
                </div>
              </article>
            </div>
          </section>
        </div>

        <aside class="battle-log" aria-label="Battle log">
          <h2>Battle Log</h2>
          <ol>
            <li v-for="(logEntry, index) in battleLog" :key="`${index}-${logEntry}`">
              {{ logEntry }}
            </li>
          </ol>
        </aside>
      </div>

      <div v-if="winner" class="result-panel" role="status" aria-live="polite">
        <p>Battle Result</p>
        <strong>{{ winner }}</strong>
      </div>
    </template>

    <div v-else class="team-detail-page">
      <section class="team-panel" aria-labelledby="player-team-title">
        <h2 id="player-team-title">Player Team</h2>
        <div class="monster-grid">
          <MonsterCard v-for="monster in playerMonsters" :key="monster.Name" :monster="monster" />
        </div>
      </section>

      <section class="team-panel" aria-labelledby="enemy-team-title">
        <h2 id="enemy-team-title">Enemy Team</h2>
        <div class="monster-grid">
          <MonsterCard v-for="monster in enemyMonsters" :key="monster.Name" :monster="monster" />
        </div>
      </section>
    </div>

    <div class="battle-actions">
      <button class="primary-button" type="button" @click="resetBattle">Restart Battle</button>
      <button class="back-button" type="button" @click="$emit('back')">Back to Prepare</button>
    </div>
  </section>
</template>

<style scoped>
.battle-scene {
  align-content: start;
  gap: 24px;
}

.battle-header {
  display: flex;
  width: min(100%, 1480px);
  align-items: end;
  justify-content: space-between;
  gap: 16px;
}

.battle-status {
  margin: 0;
  color: #fff6d5;
  font-weight: 900;
}

.header-actions .primary-button {
  padding: 0 24px;
}

.battle-view {
  display: grid;
  width: min(100%, 1480px);
  grid-template-columns: minmax(0, 1fr) minmax(250px, 330px);
  gap: 18px;
  align-items: stretch;
}

.sprite-arena {
  display: grid;
  min-width: 0;
  gap: 18px;
  padding: 18px;
  border-radius: 8px;
  background:
    linear-gradient(rgba(255, 246, 213, 0.06), rgba(255, 246, 213, 0)),
    rgba(9, 24, 28, 0.38);
}

.sprite-team {
  display: grid;
  gap: 10px;
  min-width: 0;
}

.sprite-team h2,
.team-panel h2,
.battle-log h2 {
  margin: 0;
  color: #f2c45f;
  font-size: 1.1rem;
}

.sprite-row {
  display: grid;
  grid-template-columns: repeat(6, minmax(86px, 1fr));
  gap: 10px;
}

.sprite-slot {
  display: grid;
  justify-items: center;
  gap: 8px;
  min-width: 0;
  padding: 10px;
  border: 1px solid rgba(255, 246, 213, 0.16);
  border-radius: 8px;
  background: rgba(255, 246, 213, 0.05);
}

.sprite-slot.active {
  border-color: rgba(242, 196, 95, 0.82);
  box-shadow: inset 0 0 0 1px rgba(242, 196, 95, 0.25);
}

.sprite-slot.fainted {
  opacity: 0.42;
}

.sprite-name {
  width: 100%;
  overflow: hidden;
  color: #fff6d5;
  font-size: 0.76rem;
  font-weight: 900;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.sprite {
  position: relative;
  display: flex;
  width: min(12vw, 92px);
  min-width: 62px;
  aspect-ratio: 1;
  align-items: center;
  justify-content: center;
  gap: 12px;
  border: 3px solid #fff6d5;
  box-shadow:
    0 14px 22px rgba(0, 0, 0, 0.22),
    inset 0 -12px 0 rgba(0, 0, 0, 0.12);
  transition: filter 120ms ease;
}

.player-sprite {
  border-radius: 46% 54% 42% 58%;
  background: linear-gradient(155deg, #80d2a9, #40857a);
}

.enemy-sprite {
  border-radius: 58% 42% 54% 46%;
  background: linear-gradient(155deg, #ef7d4c, #9e3941);
}

.sprite-eye {
  width: 12px;
  height: 12px;
  border-radius: 999px;
  background: #1f2426;
}

.player-sprite.attacking {
  animation: player-attack 360ms ease;
}

.enemy-sprite.attacking {
  animation: enemy-attack 360ms ease;
}

.sprite.hit {
  animation: take-hit 360ms ease;
}

.damage-indicator {
  position: absolute;
  top: -18px;
  left: 50%;
  color: #ffef8a;
  font-size: 1rem;
  font-weight: 900;
  text-shadow: 0 2px 0 rgba(0, 0, 0, 0.35);
  transform: translateX(-50%);
  animation: damage-float 760ms ease-out forwards;
  pointer-events: none;
}

.sprite-hp {
  width: 100%;
  height: 8px;
  overflow: hidden;
  border-radius: 999px;
  background: rgba(255, 246, 213, 0.16);
}

.sprite-hp span {
  display: block;
  height: 100%;
  border-radius: inherit;
  background: linear-gradient(90deg, #65d18d, #f2c45f);
  transition: width 180ms ease;
}

.arena-center {
  display: grid;
  min-height: 48px;
  place-items: center;
  color: #f2c45f;
  font-size: 2rem;
  font-weight: 900;
}

.battle-log {
  display: grid;
  grid-template-rows: auto minmax(0, 1fr);
  gap: 12px;
  min-height: 460px;
  max-height: 640px;
  min-width: 0;
  padding: 16px;
  border-radius: 8px;
  background: rgba(9, 24, 28, 0.38);
  text-align: left;
}

.battle-log ol {
  display: grid;
  align-content: start;
  gap: 10px;
  min-height: 0;
  margin: 0;
  overflow-y: auto;
  padding-left: 20px;
  padding-right: 6px;
}

.battle-log li {
  color: rgba(255, 246, 213, 0.88);
  font-size: 0.84rem;
  font-weight: 800;
  line-height: 1.35;
}

.result-panel {
  display: grid;
  gap: 4px;
  width: min(100%, 420px);
  padding: 16px;
  border: 2px solid rgba(242, 196, 95, 0.52);
  border-radius: 8px;
  background: rgba(9, 24, 28, 0.5);
}

.result-panel p,
.result-panel strong {
  margin: 0;
}

.result-panel p {
  color: #f2c45f;
  font-size: 0.8rem;
  font-weight: 900;
  letter-spacing: 0.12em;
  text-transform: uppercase;
}

.result-panel strong {
  color: #fff6d5;
  font-size: 1.35rem;
}

.team-detail-page {
  display: grid;
  width: min(100%, 1480px);
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 18px;
  align-items: start;
}

.team-panel {
  display: grid;
  gap: 12px;
  min-width: 0;
  padding: 16px;
  border-radius: 8px;
  background: rgba(9, 24, 28, 0.38);
}

.monster-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 12px;
}

.battle-actions {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 12px;
}

.battle-actions .primary-button {
  padding: 0 28px;
}

@media (max-width: 1180px) {
  .battle-header,
  .battle-view,
  .team-detail-page {
    grid-template-columns: 1fr;
  }

  .battle-header {
    display: grid;
    justify-items: center;
  }

  .battle-log {
    min-height: 260px;
    max-height: 320px;
  }
}

@media (max-width: 760px) {
  .sprite-row {
    grid-template-columns: repeat(3, minmax(82px, 1fr));
  }

  .monster-grid {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 430px) {
  .sprite-row {
    grid-template-columns: repeat(2, minmax(82px, 1fr));
  }
}

@keyframes player-attack {
  35% {
    transform: translateY(18px) scale(1.06);
  }

  100% {
    transform: translateY(0) scale(1);
  }
}

@keyframes enemy-attack {
  35% {
    transform: translateY(-18px) scale(1.06);
  }

  100% {
    transform: translateY(0) scale(1);
  }
}

@keyframes take-hit {
  20% {
    transform: translateX(-7px);
    filter: brightness(1.35);
  }

  45% {
    transform: translateX(7px);
  }

  100% {
    transform: translateX(0);
    filter: brightness(1);
  }
}

@keyframes damage-float {
  0% {
    opacity: 0;
    transform: translate(-50%, 8px) scale(0.9);
  }

  20% {
    opacity: 1;
  }

  100% {
    opacity: 0;
    transform: translate(-50%, -30px) scale(1.12);
  }
}
</style>
