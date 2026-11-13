<script setup>
import confetti from "canvas-confetti";
import { ref, computed } from "vue";
import ColorPalette from "./ColorPalette.vue";
import GameRowVue from "./GameRow.vue";
import GameOverModal from "./GameOverModal.vue";
import GameStats from "./GameStats.vue";

const colors = ["red", "blue", "green", "yellow", "black", "white"];

// 1. The Mulberry32 generator function
function mulberry32(seed) {
  let t = seed >>> 0;
  return function () {
    t = (t + 0x6d2b79f5) >>> 0;
    let r = Math.imul(t ^ (t >>> 15), t | 1);
    r ^= r + Math.imul(r ^ (r >>> 7), r | 61);
    return ((r ^ (r >>> 14)) >>> 0) / 4294967296;
  };
}

// 2. Check URL for a seed, or create a random one
const params = new URLSearchParams(window.location.search);
const seedParam = params.get("seed");
let gameSeed = seedParam
  ? parseInt(seedParam, 10)
  : Math.floor(Math.random() * 1000000);

// 3. Create our generator and update getRandomColor
let randomNumberGenerator = mulberry32(gameSeed);
const getRandomColor = () =>
  colors[Math.floor(randomNumberGenerator() * colors.length)];

// 4. Your existing secretCode ref uses getRandomColor() automatically!
const secretCode = ref([
  getRandomColor(),
  getRandomColor(),
  getRandomColor(),
  getRandomColor(),
]);

const isChallengeMode = computed(() => {
  const params = new URLSearchParams(window.location.search);
  return params.has("seed");
});

const currentGuess = ref([null, null, null, null]);
const allGuesses = ref([]);
const allFeedback = ref([]); // whitePegs and blackPegs for each guess
const activeRow = ref(1);
// end-of-game modal: { type: 'win' | 'lose', text } or null when hidden
const gameMessage = ref(null);
// scores holds previous scores; need '[]' rather than [] because JSON.parse expects a string
// if localStorage is null, or an empty string, it falls back to the empty array
const scores = ref(JSON.parse(localStorage.getItem("scores") || "[]"));

function shareGame() {
  // 1. Build the full URL using the current website address and our gameSeed
  const shareUrl = `${window.location.origin}${window.location.pathname}?seed=${gameSeed}`;

  // 2. Copy it to the player's clipboard
  navigator.clipboard
    .writeText(shareUrl)
    .then(() => {
      alert("Challenge link copied to clipboard! Send it to a friend.");
    })
    .catch((err) => {
      console.error("Failed to copy: ", err);
    });
}

function clearStats() {
  localStorage.removeItem("scores");
  scores.value = [];
}

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

function handleClearSlot(row, slot) {
  if (row === activeRow.value - 1) {
    currentGuess.value[slot] = null;
  }
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
  // if the guess is correct, celebrate with the win modal, and add # guesses it took player to scores array
  if (blackPegs === 4) {
    celebrate();
    gameMessage.value = {
      type: "win",
      text: "Congratulations! You've cracked the code!",
    };
    scores.value.push(allGuesses.value.length); //allGuesses.values.length==number of turns
    localStorage.setItem("scores", JSON.stringify(scores.value));
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
  // Generate a brand new random seed for this new game
  gameSeed = Math.floor(Math.random() * 1000000);
  // Re-initialize our RNG generator with the new seed
  randomNumberGenerator = mulberry32(gameSeed);
  // reset the game state
  currentGuess.value = [null, null, null, null];
  allGuesses.value = [];
  allFeedback.value = [];
  activeRow.value = 1;
  // generate new secretCode
  secretCode.value = [
    getRandomColor(),
    getRandomColor(),
    getRandomColor(),
    getRandomColor(),
  ];
  // Clear the ?seed= from the browser URL bar without reloading the page
  window.history.pushState({}, "", window.location.pathname);
}
</script>

<template>
  <main class="game">
    <h1 class="title">Mastermind</h1>

    <div class="board">
      <!-- Row 10 at top of screen. For each row...  -->
      <!-- <div class="row" v-for="n in [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]" :key="n"> -->
      <!-- handleClearSlot needs to know both the row and the slot in the row to clear  -->
      <GameRowVue
        v-for="n in [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]"
        :rowNumber="n"
        :guess="
          n === activeRow
            ? currentGuess
            : n < activeRow
              ? allGuesses[n - 1]
              : [null, null, null, null]
        "
        :feedback="n < activeRow ? allFeedback[n - 1] : null"
        :key="n"
        @clearSlot="(slotIndex) => handleClearSlot(n - 1, slotIndex)"
      />
    </div>

    <div class="controls">
      <ColorPalette :colors="colors" @selectColor="selectColor" />
      <button class="check" @click="checkGuess">Check</button>
      <button class="check" style="padding: 0.5rem 6rem" @click="newGame">
        New Game
      </button>
      <button class="check" @click="shareGame">Share Challenge</button>
    </div>

    <div v-if="isChallengeMode" class="challenge-banner">
      <span>🎯 Challenge Mode Active</span>
    </div>

    <!-- Statistics summary section -->
    <GameStats :scores="scores" />
    <button @click="clearStats" class="reset-btn">Reset Stats</button>

    <!-- end-of-game modal (replaces the old alert boxes) -->
    <GameOverModal
      :gameMessage="gameMessage"
      @modalClick="gameMessage = null"
    />
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

.challenge-banner {
  background-color: #e6dfd3; /* Soft earthy background */
  border: 1px solid #d4c4b0;
  color: #4a453e;
  padding: 0.5rem 1rem;
  width: 97%;
  border-radius: 0.375rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 0.875rem;
  margin-bottom: 1rem;
}

.title {
  margin: 0;
  font-size: 1.5rem;
  font-weight: 600;
}

.board {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
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

.reset-btn {
  background: transparent;
  border: 1px solid rgba(0, 0, 0, 0.2);
  color: #555;
  font-size: 0.75rem;
  padding: 0.25rem 0.5rem;
  border-radius: 0.25rem;
  cursor: pointer;
  float: right;
}
</style>
