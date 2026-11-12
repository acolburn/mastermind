<script setup>
// import { defineProps, defineEmits } from "vue";

const props = defineProps({
  gameMessage: Object,
});
const emits = defineEmits(["modalClick"]);
</script>

<template>
  <div v-if="gameMessage" class="overlay">
    <div class="modal">
      <p class="modal-text">{{ gameMessage.text }}</p>
      <button class="check" @click="$emit('modalClick')">OK</button>
    </div>
  </div>
</template>

<style scoped>
.check {
  padding: 0.5rem 2rem;
  font-size: 1rem;
  border: none;
  border-radius: 0.375rem;
  background: #333;
  color: #fff;
  cursor: pointer;
}

.overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal {
  position: relative;
  overflow: hidden;
  background: #fff;
  border-radius: 0.75rem;
  padding: 1.5rem 2rem;
  max-width: 20rem;
  text-align: center;
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.35);
}

.modal-text {
  margin: 0 0 1rem;
}
</style>
