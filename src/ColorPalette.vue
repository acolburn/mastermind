<script setup>
import { defineProps, defineEmits } from "vue";

defineProps({
  colors: {
    type: Array,
    required: true,
  },
});

const emits = defineEmits(["selectColor"]);
</script>

<template>
  <div class="palette">
    <button
      v-for="col in colors"
      :key="col"
      class="color"
      :class="col"
      @click="$emit('selectColor', col)"
    ></button>
  </div>
</template>

<style scoped>
.palette {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 0.5rem;
}

.color {
  width: 2.25rem;
  aspect-ratio: 1;
  border-radius: 50%;
  border: 2px solid rgba(0, 0, 0, 0.2);
  cursor: pointer;
}

.color.red {
  background: #e74c3c;
}
.color.black {
  background: #000000;
}
.color.yellow {
  background: #f1c40f;
}
.color.green {
  background: #2ecc71;
}
.color.blue {
  background: #3498db;
}
.color.white {
  background: #ffffff;
}
</style>
