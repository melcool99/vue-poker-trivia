<script lang="ts" setup>
import { computed, defineProps } from 'vue'

const props = defineProps({
  rank: String,
  suit: String,
  showCards: Boolean
})

const suitSymbol = computed(() => {
  const suit = props.suit
  switch (suit) {
    case 'C':
      return '♣'
    case 'S':
      return '♠'
    case 'H':
      return '♥'
    case 'D':
      return '♦'
    default:
      return ''
  }
})
const suitClass = computed(() => {
  const suit = props.suit
  if (suit === 'C' || suit === 'S') {
    return 'text-dark'
  } else {
    return 'text-danger'
  }
})
</script>

<template>
  <div :class="suitClass" class="poker-card">
    <div class="poker-card-rank">{{ rank }}</div>
    <div class="poker-card-suit">{{ suitSymbol }}</div>
  </div>
</template>

<style scoped>
.poker-card {
  position: relative;
  box-shadow: 0 0 5px var(--dark-color);
  border-radius: 5px;
  width: 60px;
  height: 100px;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 10px;
  background-color: var(--light-color);
  margin: 0 5px;
}

.poker-card-rank {
  font-size: 60px;
}

.poker-card-suit {
  font-size: 40px;
}

.text-dark {
  color: var(--dark-color);
}
.text-danger {
  color: var(--danger-color);
}
</style>
