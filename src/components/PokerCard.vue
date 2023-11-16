<script lang="ts" setup>
import { computed, defineProps } from 'vue'

const props = defineProps({
  rank: String,
  suit: String
})

const suitSymbol = computed(() => {
  const suit = props.suit
  switch (suit) {
    case 'C':
      return '♣'
    case 'S':
      return '♠'
    case 'H':
      return '❤'
    case 'D':
      return '♦'
    default:
      return ''
  }
})
</script>

<template>
  <div :class="suitSymbol" class="poker-card">
    <div class="poker-card__rank">{{ rank }}</div>
    <div class="poker-card__suit">{{ suitSymbol }}</div>
  </div>
</template>

<style scoped>
.poker-card {
  position: relative;
  border: 1px solid var(--info-color);
  border-radius: 5px;
  width: 80px;
  height: 120px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  background-color: var(--light-color);
}

.poker-card__rank {
  font-size: 80px;
}

.poker-card__suit {
  font-size: 80px;
}
</style>
