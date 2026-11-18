<script setup>
import { ref } from "vue";

const props = defineProps({
  isOpen: Boolean,
});

const emit = defineEmits(["addScore", "close"]);

const scoreInput = ref("");
const error = ref("");

function handleAddScore() {
  error.value = "";
  
  // Validate input - convert to string first to handle number type
  const inputValue = String(scoreInput.value);
  const score = parseInt(inputValue, 10);
  
  if (inputValue.trim() === "") {
    error.value = "Please enter a score";
    return;
  }
  
  if (isNaN(score)) {
    error.value = "Please enter a valid number";
    return;
  }
  
  if (score < 1 || score > 10) {
    error.value = "Score must be between 1 and 10";
    return;
  }
  
  // Emit the score and reset
  emit("addScore", score);
  scoreInput.value = "";
  error.value = "";
}

function handleClose() {
  scoreInput.value = "";
  error.value = "";
  emit("close");
}

function handleKeyDown(event) {
  if (event.key === "Enter") {
    event.preventDefault();
    handleAddScore();
  } else if (event.key === "Escape") {
    handleClose();
  }
}
</script>

<template>
  <div v-if="isOpen" class="modal-overlay" @click="handleClose">
    <div class="modal" @click.stop>
      <h2>Add a Score</h2>
      <p class="instructions">Enter the number of guesses it took to solve (1-10):</p>
      
      <input
        v-model="scoreInput"
        type="number"
        min="1"
        max="10"
        placeholder="e.g., 5"
        @keydown="handleKeyDown"
        class="score-input"
        autofocus
      />
      
      <div v-if="error" class="error-message">{{ error }}</div>
      
      <div class="modal-buttons">
        <button @click="handleAddScore" class="add-btn">Add Score</button>
        <button @click="handleClose" class="cancel-btn">Cancel</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal {
  background-color: #d4c4b0;
  border-radius: 0.5rem;
  padding: 2rem;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.3);
  max-width: 20rem;
  width: 90%;
  border: 1px solid rgba(0, 0, 0, 0.2);
}

h2 {
  margin: 0 0 1rem 0;
  font-size: 1.3rem;
  color: #333;
  text-align: center;
}

.instructions {
  margin: 0 0 1rem 0;
  color: #555;
  text-align: center;
  font-size: 0.95rem;
}

.score-input {
  width: 100%;
  padding: 0.75rem;
  font-size: 1rem;
  border: 1px solid rgba(0, 0, 0, 0.2);
  border-radius: 0.375rem;
  box-sizing: border-box;
  margin-bottom: 1rem;
}

.score-input:focus {
  outline: none;
  border-color: #333;
  box-shadow: 0 0 0 2px rgba(51, 51, 51, 0.1);
}

.error-message {
  color: #c74646;
  font-size: 0.85rem;
  margin-bottom: 1rem;
  text-align: center;
}

.modal-buttons {
  display: flex;
  gap: 0.75rem;
  justify-content: center;
}

.add-btn,
.cancel-btn {
  padding: 0.5rem 1.5rem;
  font-size: 0.95rem;
  border: none;
  border-radius: 0.375rem;
  cursor: pointer;
  font-weight: 500;
}

.add-btn {
  background-color: #333;
  color: #fff;
}

.add-btn:hover {
  background-color: #555;
}

.cancel-btn {
  background-color: transparent;
  border: 1px solid #333;
  color: #333;
}

.cancel-btn:hover {
  background-color: #c8bba9;
}
</style>
