<template>
  <div v-if="isVisible" class="modal">
    <div class="modal-content">
      <span class="close" @click="closeModal">&times;</span>
      <slot></slot>
    </div>
  </div>
</template>

<script setup>
import { ref, defineExpose } from 'vue'

const isVisible = ref(false)

const closeModal = () => {
  isVisible.value = false
}

const showModal = () => {
  isVisible.value = true
}

defineExpose({
  isVisible,
  showModal,
  closeModal
})
</script>

<style scoped>
.modal {
  display: block;
  position: fixed;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background-color: var(--black);
  z-index: 1000;
}

.modal-content {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: var(--black);
  padding: 0;
  border: none;
  width: 80%;
  height: 80%;
  display: flex;
  justify-content: center;
  align-items: center;
}

.close {
  position: absolute;
  top: 10px;
  right: 20px;
  color: #aaa;
  font-size: 40px;
  font-weight: bold;
}

.close:hover,
.close:focus {
  color: white;
  text-decoration: none;
  cursor: pointer;
}

.video-container {
  width: 100%;
  height: 100%;
}

iframe {
  width: 100%;
  height: 100%;
  border: none;
}
</style>
