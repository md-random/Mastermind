<template>
  <div>
    <h1>Mastermind</h1>
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

const secretCode = ref<string[]>(
  Array.from(
    { length: codeLength },
    () => colors[Math.floor(Math.random() * colors.length)]
  )
)

const guesses = ref<string[][]>([])
const currentGuess = ref<string[]>([])

const gameOver = computed(
  () => guesses.value.length >= 10 || guesses.value.some(isCorrectGuess)
)

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
</script>
