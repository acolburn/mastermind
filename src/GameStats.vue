<script setup>
import { computed } from "vue";

const props = defineProps({
  scores: Array,
});

const gamesPlayed = computed(() => props.scores.length);

const avgScore = computed(() => {
  if (props.scores.length === 0) {
    return "0.0";
  }

  let total = 0;
  for (const score of props.scores) {
    total = total + score;
  }
  // props can be used directly, computed values need *.value
  return (total / gamesPlayed.value).toFixed(1);
});

// Compute frequency distribution of scores
const scoreDistribution = computed(() => {
  // initialize
  const distribution = {
    1: 0,
    2: 0,
    3: 0,
    4: 0,
    5: 0,
    6: 0,
    7: 0,
    8: 0,
    9: 0,
    10: 0,
  };
  for (const score of props.scores) {
    distribution[score]++;
  }
  return distribution;
});

// return number of games for a given score, e.g., how many games solved in 4 turns, 5 turns, etc.
function getGameCount(score) {
  return scoreDistribution.value[score] || 0;
}

function getBarWidth(score) {
  // counts holds array's values, e.g., [#games solved in 1, #games solved in 2, etc.]
  const counts = Object.values(scoreDistribution.value);
  // find largest value in counts; if everything is 0, maxCount = 1
  const maxCount = Math.max(...counts, 1); // Avoid division by zero if all counts are 0
  const count = getGameCount(score);
  return (count / maxCount) * 100;
}
</script>

<template>
  <div class="statistics">
    <h2 class="stats-title">Summary</h2>

    <div class="stats-grid">
      <div class="stat-card">
        <div class="stat-label">Games Played</div>
        <div class="stat-value">{{ gamesPlayed }}</div>
      </div>

      <div class="stat-card">
        <div class="stat-label">Avg Score</div>
        <div class="stat-value">{{ avgScore }}</div>
      </div>
    </div>

    <div class="recent-scores">
      <p v-if="scores.length === 0" style="text-align: center">
        Complete your first game to see your stats!
      </p>
      <div v-else class="histogram">
        <div class="histogram-row" v-for="n in 10" :key="n">
          <div class="guess-label">{{ n }}</div>
          <div class="bar-container">
            <div class="bar" :style="{ width: getBarWidth(n) + '%' }">
              <span class="bar-count">{{ getGameCount(n) }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.statistics {
  width: 97%;
  margin-top: 1rem;
  padding: 1rem;
  background-color: #d4c4b0;
  border-radius: 0.5rem;
  border: 1px solid rgba(0, 0, 0, 0.1);
}

.stats-title {
  margin: 0 0 1rem 0;
  font-size: 1.1rem;
  text-align: center;
  color: #333;
}

.stats-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
  margin-bottom: 1rem;
}

.stat-card {
  background-color: #c8bba9;
  padding: 0.75rem;
  border-radius: 0.375rem;
  text-align: center;
  border: 1px solid rgba(0, 0, 0, 0.05);
}

.stat-label {
  font-size: 0.85rem;
  color: #555;
  margin-bottom: 0.5rem;
}

.stat-value {
  font-size: 1.75rem;
  font-weight: bold;
  color: #333;
}

.recent-scores {
  border-top: 1px solid rgba(0, 0, 0, 0.1);
  padding-top: 1rem;
  width: 100%;
}

.histogram {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.histogram-row {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.guess-label {
  font-size: 0.9rem;
  color: #333;
  font-weight: bold;
  min-width: 1.5rem;
  text-align: center;
}

.bar-container {
  flex: 1;
  height: 1.5rem;
  background-color: #c8bba9;
  border-radius: 0.25rem;
  border: 1px solid rgba(0, 0, 0, 0.05);
  overflow: hidden;
}

.bar {
  height: 100%;
  background-color: #a89968;
  display: flex;
  align-items: center;
  justify-content: flex-end;
  padding-right: 0.5rem;
  transition: width 0.2s ease;
}

.bar-count {
  font-size: 0.85rem;
  font-weight: bold;
  color: #fff;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.3);
}

.recent-title {
  margin: 0;
  font-size: 0.95rem;
  color: #333;
}

.scores-list {
  display: flex;
  flex-direction: row;
  justify-content: center;
  gap: 0.5rem;
}

.score-item {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 0.34rem 0.51rem;
  background-color: #c8bba9;
  border-radius: 0.25rem;
  border: 1px solid rgba(0, 0, 0, 0.05);
  font-size: 1rem;
  font-weight: bold;
  color: #333;
  min-width: 2.5rem;
}
</style>
