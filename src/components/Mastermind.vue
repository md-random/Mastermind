<template>
  <div>
    <div class="version-align">version 1.0</div>

    <h1>Mastermind</h1>
    <button @click="startNewGame">New Game</button>
    <p>Guesses: {{ guessCount }} / 10</p>
    <div v-if="!gameOver">
      <div>
        <button
          v-for="color in colors"
          :key="color"
          :style="{ backgroundColor: color }"
          @click="addColor(color)"
        >
          {{ color }}
        </button>
      </div>
      <div>
        Current guess:
        <span
          v-for="(color, index) in currentGuess"
          :key="index"
          :style="{ backgroundColor: color }"
        >
          {{ color }}
        </span>
      </div>
      <button @click="removeLastColor">Remove Last</button>
      <button @click="submitGuess">Submit Guess</button>
    </div>
    <div v-else>
      <h2>Game Over!</h2>
      <p>The secret code was: {{ secretCode.join(', ') }}</p>
    </div>
    <div>
      <h3>Guesses:</h3>
      <ul>
        <li v-for="(guess, index) in guesses" :key="index">
          {{ guess.join(', ') }} - Correct: {{ getFeedback(guess).correct }},
          Misplaced: {{ getFeedback(guess).misplaced }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

const colors = ['red', 'blue', 'green', 'yellow', 'purple', 'orange']
const codeLength = 4

const secretCode = ref<string[]>([])
const guesses = ref<string[][]>([])
const currentGuess = ref<string[]>([])
const guessCount = ref(0)

const gameOver = computed(
  () => guessCount.value >= 10 || guesses.value.some(isCorrectGuess)
)

const generateSecretCode = () => {
  return Array.from(
    { length: codeLength },
    () => colors[Math.floor(Math.random() * colors.length)]
  )
}

const startNewGame = () => {
  secretCode.value = generateSecretCode()
  guesses.value = []
  currentGuess.value = []
  guessCount.value = 0
}

const addColor = (color: string) => {
  if (currentGuess.value.length < codeLength) {
    currentGuess.value.push(color)
  }
}

const removeLastColor = () => {
  currentGuess.value.pop()
}

const submitGuess = () => {
  if (currentGuess.value.length === codeLength) {
    guesses.value.push([...currentGuess.value])
    currentGuess.value = []
    guessCount.value++
  }
}

const isCorrectGuess = (guess: string[]): boolean => {
  return guess.every((color, index) => color === secretCode.value[index])
}

const getFeedback = (
  guess: string[]
): { correct: number; misplaced: number } => {
  let correct = 0
  let misplaced = 0
  const codeCopy = [...secretCode.value]

  guess.forEach((color, index) => {
    if (color === codeCopy[index]) {
      correct++
      codeCopy[index] = ''
    }
  })

  guess.forEach((color, index) => {
    if (color !== secretCode.value[index] && codeCopy.includes(color)) {
      misplaced++
      codeCopy[codeCopy.indexOf(color)] = ''
    }
  })

  return { correct, misplaced }
}

// Initialize the game
startNewGame()
</script>
<style scoped>
.version-align {
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 14px;
  color: #4169e1; /* Royal Blue */
  font-weight: bold;
}
</style>
