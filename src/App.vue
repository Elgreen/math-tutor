<template>
  <main class="container">
    <header class="hero">
      <h1>Multiplication Table Trainer</h1>
      <p>
        Pick a number from 1 to 10, study its multiplication table, and then practice with five
        random problems.
      </p>
    </header>

    <section class="card">
      <h2>Step 1. Choose a number</h2>
      <p class="hint">Tap a button to pick the number X.</p>
      <div class="number-grid">
        <button
          v-for="number in numbers"
          :key="number"
          type="button"
          class="number-button"
          :class="{ active: number === selectedNumber }"
          @click="chooseNumber(number)"
        >
          {{ number }}
        </button>
      </div>
    </section>

    <section v-if="showTable" class="card">
      <div class="card-header">
        <h2>Step 2. Multiplication table for {{ selectedNumber }}</h2>
        <button type="button" class="secondary" @click="startPractice">Start practice</button>
      </div>
      <table class="multiplication-table">
        <tbody>
          <tr v-for="factor in numbers" :key="`row-${factor}`">
            <td>{{ selectedNumber }}</td>
            <td>×</td>
            <td>{{ factor }}</td>
            <td>=</td>
            <td>{{ selectedNumber * factor }}</td>
          </tr>
        </tbody>
      </table>
    </section>

    <section v-if="showPractice" class="card">
      <div class="card-header">
        <h2>Step 3. Practice</h2>
        <button type="button" class="secondary" @click="showTheoryAgain">Back to the table</button>
      </div>
      <p class="hint">
        Fill in the answers for five problems. The order of the factors may change, but the result
        stays the same!
      </p>
      <form class="practice-form" @submit.prevent="checkAnswers">
        <div
          v-for="(problem, index) in practiceProblems"
          :key="problem.id"
          class="practice-row"
        >
          <span class="expression">{{ problem.left }} × {{ problem.right }} =</span>
          <input
            v-model="answers[index]"
            type="number"
            min="0"
            required
            :aria-label="`Answer for ${problem.left} × ${problem.right}`"
          />
          <span v-if="hasResults" class="badge" :class="results[index].isCorrect ? 'badge-success' : 'badge-error'">
            {{ results[index].isCorrect ? 'Correct' : `Expected: ${results[index].correct}` }}
          </span>
        </div>

        <div class="actions">
          <button type="submit">Check answers</button>
          <button type="button" class="secondary" @click="generatePractice">New problems</button>
          <button type="button" class="link" @click="reset">Choose another number</button>
        </div>
      </form>

      <div v-if="hasResults" class="summary" :class="allCorrect ? 'summary-success' : 'summary-info'">
        <p v-if="allCorrect">Great job! All answers are correct 🎉</p>
        <template v-else>
          <p>There are mistakes, try again.</p>
          <ul>
            <li v-for="result in incorrectResults" :key="result.id">
              {{ result.left }} × {{ result.right }} = {{ result.correct }}, your answer:
              {{ result.userAnswer ?? '—' }}
            </li>
          </ul>
        </template>
      </div>
    </section>
  </main>
</template>

<script setup>
import { computed, ref } from 'vue';

const numbers = Array.from({ length: 10 }, (_, index) => index + 1);
const selectedNumber = ref(null);
const showTable = ref(false);
const showPractice = ref(false);
const practiceProblems = ref([]);
const answers = ref([]);
const results = ref([]);
const hasResults = ref(false);
let problemCounter = 0;

const allCorrect = computed(() => hasResults.value && results.value.every((item) => item.isCorrect));
const incorrectResults = computed(() => results.value.filter((item) => !item.isCorrect));

function chooseNumber(number) {
  if (selectedNumber.value === number && showTable.value) {
    return;
  }

  selectedNumber.value = number;
  showTable.value = true;
  showPractice.value = false;
  hasResults.value = false;
}

function generatePractice() {
  if (!selectedNumber.value) {
    return;
  }

  const availableMultipliers = [...numbers];
  const pickedMultipliers = [];

  while (pickedMultipliers.length < 5 && availableMultipliers.length) {
    const randomIndex = Math.floor(Math.random() * availableMultipliers.length);
    pickedMultipliers.push(availableMultipliers.splice(randomIndex, 1)[0]);
  }

  practiceProblems.value = pickedMultipliers.map((multiplier) => {
    const swapOperands = Math.random() > 0.5;
    const left = swapOperands ? multiplier : selectedNumber.value;
    const right = swapOperands ? selectedNumber.value : multiplier;

    return {
      id: ++problemCounter,
      left,
      right,
      correct: selectedNumber.value * multiplier,
    };
  });

  answers.value = practiceProblems.value.map(() => '');
  results.value = practiceProblems.value.map((problem) => ({ ...problem, userAnswer: null, isCorrect: false }));
  hasResults.value = false;
}

function startPractice() {
  showTable.value = false;
  showPractice.value = true;
  generatePractice();
}

function showTheoryAgain() {
  showPractice.value = false;
  showTable.value = true;
  hasResults.value = false;
}

function checkAnswers() {
  results.value = practiceProblems.value.map((problem, index) => {
    const userValue = answers.value[index];
    const parsed = userValue === '' ? null : Number(userValue);
    const isCorrect = parsed === problem.correct;

    return {
      ...problem,
      userAnswer: parsed,
      isCorrect,
    };
  });

  hasResults.value = true;
}

function reset() {
  selectedNumber.value = null;
  showTable.value = false;
  showPractice.value = false;
  hasResults.value = false;
  answers.value = [];
  results.value = [];
  practiceProblems.value = [];
}
</script>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.hero {
  text-align: center;
  background: white;
  border-radius: 24px;
  padding: 32px 24px;
  box-shadow: 0 18px 40px -32px rgba(15, 23, 42, 0.4);
}

.hero h1 {
  margin: 0 0 12px;
  font-size: 2rem;
}

.hero p {
  margin: 0;
  line-height: 1.6;
}

.card {
  background: white;
  border-radius: 20px;
  padding: 24px;
  box-shadow: 0 10px 32px -24px rgba(15, 23, 42, 0.35);
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.card-header {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 12px;
  justify-content: space-between;
}

.card h2 {
  margin: 0;
  font-size: 1.4rem;
}

.hint {
  margin: 0;
  color: #64748b;
}

.number-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(64px, 1fr));
  gap: 12px;
}

.number-button {
  border: none;
  border-radius: 14px;
  padding: 12px 0;
  font-size: 1.1rem;
  background: #e0f2fe;
  color: #0f172a;
  cursor: pointer;
  transition: transform 0.15s ease, box-shadow 0.15s ease, background 0.15s ease;
}

.number-button:hover {
  transform: translateY(-2px);
  box-shadow: 0 12px 20px -16px rgba(37, 99, 235, 0.6);
}

.number-button.active {
  background: linear-gradient(135deg, #38bdf8, #2563eb);
  color: white;
  box-shadow: 0 12px 24px -12px rgba(37, 99, 235, 0.6);
}

.secondary {
  border: 1px solid #94a3b8;
  background: transparent;
  color: #334155;
  padding: 10px 16px;
  border-radius: 12px;
  cursor: pointer;
  transition: background 0.15s ease, color 0.15s ease, border-color 0.15s ease;
}

.secondary:hover {
  background: #e2e8f0;
}

.link {
  background: none;
  border: none;
  color: #2563eb;
  cursor: pointer;
  text-decoration: underline;
  padding: 8px 0;
}

button[type='submit'] {
  background: linear-gradient(135deg, #2563eb, #7c3aed);
  color: white;
  border: none;
  padding: 12px 20px;
  border-radius: 12px;
  cursor: pointer;
  font-size: 1rem;
  transition: transform 0.15s ease, box-shadow 0.15s ease;
}

button[type='submit']:hover {
  transform: translateY(-1px);
  box-shadow: 0 16px 24px -18px rgba(76, 29, 149, 0.55);
}

.multiplication-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 1.1rem;
}

.multiplication-table td {
  padding: 8px;
  border-bottom: 1px solid #e2e8f0;
  text-align: center;
}

.practice-form {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.practice-row {
  display: flex;
  align-items: center;
  gap: 12px;
  flex-wrap: wrap;
}

.expression {
  min-width: 160px;
  font-weight: 600;
}

.practice-row input {
  width: 120px;
  padding: 10px;
  border: 1px solid #cbd5f5;
  border-radius: 10px;
  background: #f8fafc;
}

.actions {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  align-items: center;
}

.badge {
  border-radius: 999px;
  padding: 6px 12px;
  font-size: 0.9rem;
  font-weight: 600;
}

.badge-success {
  background: rgba(34, 197, 94, 0.18);
  color: #0f5132;
}

.badge-error {
  background: rgba(248, 113, 113, 0.18);
  color: #7f1d1d;
}

.summary {
  border-radius: 16px;
  padding: 16px;
  border: 1px solid transparent;
  background: #eff6ff;
}

.summary-success {
  background: rgba(34, 197, 94, 0.16);
  border-color: rgba(34, 197, 94, 0.6);
}

.summary-info {
  background: rgba(59, 130, 246, 0.12);
  border-color: rgba(59, 130, 246, 0.4);
}

.summary ul {
  margin: 8px 0 0 18px;
}

@media (max-width: 600px) {
  .expression {
    min-width: 0;
  }

  .practice-row input {
    flex: 1;
  }
}
</style>
