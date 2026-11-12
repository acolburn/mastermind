<script setup>
// import { defineProps, defineEmits } from "vue";

const props = defineProps({
  rowNumber: Number,
  guess: Array,
  feedback: Object,
});
const emits = defineEmits(["clearSlot"]);
</script>

<template>
  <div class="row">
    <!-- print the row number -->
    <span class="row-number">{{ rowNumber }}</span>
    <!-- draw a button for each item in guess -->
    <div class="holes">
      <button
        v-for="(color, index) in guess"
        class="hole"
        :key="index"
        :style="{ backgroundColor: color }"
        @click="$emit('clearSlot', index)"
      ></button>
    </div>
    <!-- draw feedback pegs -->
    <div class="feedback">
      <!-- Get blackPegs if you can. If there, use it. But if it evaluates to -->
      <!--  undefined (or any falsy value), fall back to 0 instead. -->
      <span
        class="peg"
        v-for="i in feedback?.blackPegs || 0"
        :key="'black' + i"
        style="background-color: black"
      ></span>
      <span
        class="peg"
        v-for="i in feedback?.whitePegs || 0"
        :key="'white' + i"
        style="background-color: white"
      ></span>
      <span
        class="peg"
        v-for="i in 4 - (feedback?.blackPegs || 0) - (feedback?.whitePegs || 0)"
        :key="'blank' + i"
      ></span>
    </div>
  </div>
</template>

<style scoped>
.row {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.row-number {
  width: 1.25rem;
  text-align: right;
  font-size: 0.8rem;
  color: #888;
}

.holes {
  flex: 1;
  display: flex;
  justify-content: space-between;
  gap: 0.5rem;
}

.hole {
  width: clamp(2rem, 12vw, 2.75rem);
  aspect-ratio: 1;
  border-radius: 50%;
  border: 2px solid #666;
  background: #d8d8d8;
  cursor: pointer;
}

.feedback {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 0.25rem;
}

.peg {
  width: 0.75rem;
  aspect-ratio: 1;
  border-radius: 50%;
  background: #d0d0d0;
  border: 2px solid #444;
}
</style>
