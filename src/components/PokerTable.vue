<script lang="ts" setup>
import type { Card } from '@/App.vue'
import PokerCard from './PokerCard.vue'

defineProps({
  playerCards: Array<Card>,
  communityCards: Array<Card>
})
</script>

<template>
  <div>
    <ul class="table-container">
      <PokerCard
        v-for="card in communityCards"
        :rank="card.rank"
        :suit="card.suit"
        :key="`${card.rank}${card.suit}`"
      />
    </ul>
    <div class="player-hand">
      <PokerCard
        v-for="card in playerCards"
        :rank="card.rank"
        :suit="card.suit"
        :key="`${card.rank}${card.suit}`"
      />
    </div>
  </div>
</template>

<style scoped>
.table-container {
  background-color: var(--success-color);
  border-radius: 50%;
  width: 600px;
  height: 300px;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 0 5px var(--light-color);
}
.player-hand {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: -30px;
}
</style>
