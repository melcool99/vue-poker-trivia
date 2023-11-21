<script setup lang="ts">
const visible = defineProps({
  visible: Boolean
})

const emit = defineEmits(['update:visible'])
const closeModal = () => {
  emit('update:visible', false)
}
</script>

<template>
  <div v-if="visible" class="modal-overlay" @click="closeModal">
    <div class="modal-content" @click.stop>
      <slot></slot>
    </div>
  </div>
</template>

<style scoped>
.modal-overlay {
  position: fixed;
  z-index: 1;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-content {
  display: flex;
  align-items: center;
  flex-direction: column;
  background: var(--light-color);
  padding: 20px;
  border-radius: 10px;
  font-size: 30px;
  font-weight: bold;
  margin: 10px 0;
  color: var(--dark-color);
}
</style>
