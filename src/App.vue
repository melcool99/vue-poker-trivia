<script setup lang="ts">
import { ref } from 'vue'
import PokerTable from './components/PokerTable.vue'

const playerCards = ref([
  { rank: '9', suit: 'H' },
  { rank: 'J', suit: 'C' }
])

const communityCards = ref([
  { rank: 'J', suit: 'D' },
  { rank: 'Q', suit: 'D' },
  { rank: 'K', suit: 'D' },
  { rank: 'A', suit: 'D' },
  { rank: '10', suit: 'D' }
])

const handValues = ref([
  'High Card',
  'Pair',
  'Two Pair',
  'Three of a Kind',
  'Straight',
  'Flush',
  'Full House',
  'Four of a Kind',
  'Straight Flush',
  'Royal Flush'
])

const logValue = (value: String) => {
  console.log(value)
}
</script>

<template>
  <div class="points">Points :100</div>
  <section class="hand-values-container">
    <h2>Guess your hand!</h2>
    <div class="hand-values">
      <div v-for="(value, index) in handValues" :key="index">
        <button class="btn" @click="logValue(value)">
          {{ value }}
        </button>
      </div>
    </div>
    <PokerTable :playerCards="playerCards" :communityCards="communityCards" />
  </section>
</template>

<style scoped>
.hand-values-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin: 10px 0;
}
.hand-values {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 10px;
  margin: 10px 0;
}

.points {
  font-size: 40px;
}

.btn {
  width: 150px;
  height: 35px;
  background-color: var(--light-color);
  border: none;
  border-radius: 5px;
  padding: 10px;
  margin: 0 5px;
  cursor: pointer;
  box-shadow: 0 0 10px var(--dark-color);
}
</style>
