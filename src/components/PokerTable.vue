<script lang="ts" setup>
import PokerCard from './PokerCard.vue'
</script>

<template>
  <div>
    <ul class="table-container">
      <PokerCard rank="J" suit="D" />
    </ul>
  </div>
</template>

<style scoped>
.table-container {
  background-color: var(--success-color);
  border-radius: 50%;
  width: 800px;
  height: 400px;
  padding: 10px;
}
</style>
