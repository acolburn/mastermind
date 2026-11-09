<script setup>
import confetti from "canvas-confetti";
import { ref } from "vue";
import ColorPalette from "./ColorPalette.vue";

const colors = ["red", "blue", "green", "yellow", "black", "white"];
const getRandomColor = () => colors[Math.floor(Math.random() * colors.length)];
const secretCode = ref([
  getRandomColor(),
  getRandomColor(),
  getRandomColor(),
  getRandomColor(),
]);
const currentGuess = ref([null, null, null, null]);
const allGuesses = ref([]);
const allFeedback = ref([]); // whitePegs and blackPegs for each guess
const activeRow = ref(1);
// end-of-game modal: { type: 'win' | 'lose', text } or null when hidden
const gameMessage = ref(null);

function celebrate() {
  confetti({
    particleCount: 100,
    spread: 70,
    origin: { y: 0.6 },
  });
}

function selectColor(color) {
  // find the first null in currentGuess and replace it with the selected color
  // if there is no null, index will be -1; in that case, do nothing
  const index = currentGuess.value.indexOf(null);
  if (index !== -1) {
    currentGuess.value[index] = color;
  }
}

// clear a specific slot in currentGuess (set it back to null)
function clearSlot(index) {
  currentGuess.value[index] = null;
}

function checkGuess() {
  // make sure all four slots in currentGuess are filled (not null)
  if (currentGuess.value.includes(null)) {
    alert("Please fill all slots before checking your guess.");
    return;
  }
  // save the current guess (note: we need to copy the array, not just push a reference, hence the spread operator)
  allGuesses.value.push([...currentGuess.value]);
  // make sure all the guesses continue being displayed (this is handled by the v-for in the template)
  // evaluate the guess against the secret code, black pegs for correct color and position, white pegs for correct color but wrong position
  let blackPegs = 0;
  let whitePegs = 0;
  // check for black pegs first
  let currentGuessCopy = [...currentGuess.value];
  let secretCodeCopy = [...secretCode.value];
  for (let i = 0; i < 4; i++) {
    if (currentGuessCopy[i] === secretCodeCopy[i]) {
      blackPegs++;
      // remove the matched color from currentGuessCopy to avoid counting it again for white peg loop
      currentGuessCopy[i] = null;
      // Must delete from secretCodeCopy, too, because white peg loop might look at diff unmatched guess
      // and accidentally match it against already counted match, giving false white peg.
      secretCodeCopy[i] = null;
    }
  }
  // check for white pegs
  for (let i = 0; i < 4; i++) {
    // if there's a peg that wasn't a black peg ...
    if (currentGuessCopy[i] !== null) {
      // and that peg is in the secret code (it won't be in the same position)
      if (secretCodeCopy.indexOf(currentGuessCopy[i]) !== -1) {
        whitePegs++;
        // remove the matched color from secretCodeCopy to avoid counting it again
        secretCodeCopy[secretCodeCopy.indexOf(currentGuessCopy[i])] = null;
      }
    }
  }
  // save the feedback for this guess
  allFeedback.value.push({ blackPegs, whitePegs });
  // if the guess is correct, celebrate with the win modal
  if (blackPegs === 4) {
    celebrate();
    gameMessage.value = {
      type: "win",
      text: "Congratulations! You've cracked the code!",
    };
    return;
  }
  // if the guess is incorrect and user has used all 10 guesses, show the game over modal
  if (activeRow.value === 10) {
    gameMessage.value = {
      type: "lose",
      text:
        "Game over! You've used all your guesses. The secret code was: " +
        secretCode.value.join(", "),
    };
    return;
  }
  // if the guess is incorrect, and there's turns left, move to the next row and reset currentGuess
  activeRow.value++;
  currentGuess.value = [null, null, null, null];
}

function newGame() {
  // reset the game state
  currentGuess.value = [null, null, null, null];
  allGuesses.value = [];
  allFeedback.value = [];
  activeRow.value = 1;
  secretCode.value = [
    getRandomColor(),
    getRandomColor(),
    getRandomColor(),
    getRandomColor(),
  ];
}
</script>

<template>
  <main class="game">
    <h1 class="title">Mastermind</h1>

    <div class="board">
      <!-- Row 10 at top of screen. For each row...  -->
      <div class="row" v-for="n in [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]" :key="n">
        <!-- print the row number -->
        <span class="row-number">{{ n }}</span>
        <!-- if the row is the active row, show the current guess -->
        <div v-if="n == activeRow" class="holes">
          <!-- draw a button for each item in currentGuess -->
          <button
            v-for="(color, index) in currentGuess"
            class="hole"
            :key="index"
            :style="{ backgroundColor: color }"
            @click="clearSlot(index)"
          ></button>
        </div>
        <!-- else if the row is not the active row, show the previous guesses -->
        <!-- n counts down from 10 to 1, so if n < activeRow, it means this row is a previous guess -->
        <div v-else-if="n < activeRow" class="holes">
          <button
            v-for="(color, index) in allGuesses[n - 1]"
            class="hole"
            :key="index"
            :style="{ backgroundColor: color }"
          ></button>
        </div>
        <!-- otherwise, show empty holes -->
        <div v-else class="holes">
          <button class="hole" v-for="i in 4" :key="i"></button>
        </div>

        <!-- Do basically the same thing for feedback pegs -->
        <!-- n starts at 1, allFeedback is 0-indexed, so use allFeedback[n-1]  -->
        <!-- feedback class is a grid, each <span> is a peg -->

        <div v-if="n < activeRow" class="feedback">
          <span
            class="peg"
            v-for="i in allFeedback[n - 1].blackPegs"
            :key="'black' + i"
            style="background-color: black"
          ></span>
          <span
            class="peg"
            v-for="i in allFeedback[n - 1].whitePegs"
            :key="'white' + i"
            style="background-color: white"
          ></span>
          <span
            class="peg"
            v-for="i in 4 -
            allFeedback[n - 1].blackPegs -
            allFeedback[n - 1].whitePegs"
            :key="'blank' + i"
          ></span>
        </div>

        <div v-if="n >= activeRow" class="feedback">
          <span class="peg" v-for="i in 4" :key="i"></span>
        </div>
      </div>
    </div>

    <div class="controls">
      <ColorPalette :colors="colors" @selectColor="selectColor" />
      <!-- <div class="palette">
        <button
          class="color red"
          @click="selectColor('red')"
          aria-label="red"
        ></button>
        <button
          class="color blue"
          @click="selectColor('blue')"
          aria-label="blue"
        ></button>
        <button
          class="color green"
          @click="selectColor('green')"
          aria-label="green"
        ></button>
        <button
          class="color yellow"
          @click="selectColor('yellow')"
          aria-label="yellow"
        ></button>
        <button
          class="color black"
          @click="selectColor('black')"
          aria-label="black"
        ></button>
        <button
          class="color white"
          @click="selectColor('white')"
          aria-label="white"
        ></button>
      </div> -->
      <button class="check" @click="checkGuess">Check</button>
      <button class="check" style="padding: 0.5rem 6rem" @click="newGame">
        New Game
      </button>
    </div>

    <!-- end-of-game modal (replaces the old alert boxes) -->
    <div v-if="gameMessage" class="overlay">
      <div class="modal">
        <!-- falling confetti, only when the player wins -->

        <p class="modal-text">{{ gameMessage.text }}</p>
        <button class="check" @click="gameMessage = null">OK</button>
      </div>
    </div>
  </main>
</template>

<style scoped>
.game {
  max-width: 24rem;
  margin: 0 auto;
  padding: 1rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
  background-color: #c8bba9;
}

.title {
  margin: 0;
  font-size: 1.5rem;
}

.board {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

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

.controls {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.75rem;
}

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
