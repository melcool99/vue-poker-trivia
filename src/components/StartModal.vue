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
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  display: flex;
  align-items: center;
  flex-direction: column;
  background: white;
  padding: 20px;
  border-radius: 10px;
  font-size: 30px;
  font-weight: bold;
  margin: 10px 0;
  color: var(--dark-color);
}
</style>
