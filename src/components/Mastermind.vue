<template>
  <div class="mastermind-container">
    <div class="game-board">
      <div class="version-align">version 1.4</div>

      <h1>Mastermind</h1>
      <button @click="startNewGame">New Game</button>
      <p>Guesses: {{ guessCount }} / 8</p>

      <div class="board-rows">
        <div v-for="row in 8" :key="row" class="board-row">
          <div class="guess-pegs">
            <div v-for="peg in 4" :key="peg" class="peg-hole">
              <span
                v-if="guesses[row - 1] && guesses[row - 1][peg - 1]"
                :style="{ backgroundColor: guesses[row - 1][peg - 1] }"
                class="color-peg"
              ></span>
            </div>
          </div>
          <div class="feedback-pegs">
            <div class="feedback-grid">
              <div v-for="peg in 4" :key="peg" class="feedback-peg-hole">
                <span
                  v-if="
                    guesses[row - 1] &&
                    getFeedback(guesses[row - 1]) &&
                    peg <= getFeedback(guesses[row - 1]).correct
                  "
                  class="feedback-peg correct"
                ></span>
                <span
                  v-else-if="
                    guesses[row - 1] &&
                    getFeedback(guesses[row - 1]) &&
                    peg <=
                      getFeedback(guesses[row - 1]).correct +
                        getFeedback(guesses[row - 1]).misplaced
                  "
                  class="feedback-peg misplaced"
                ></span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div v-if="!gameOver" class="controls">
        <div class="color-buttons">
          <button
            v-for="color in colors"
            :key="color"
            :style="{ backgroundColor: color }"
            @click="addColor(color)"
            class="color-peg"
          ></button>
        </div>
        <div class="current-guess">
          Current guess:
          <span
            v-for="(color, index) in currentGuess"
            :key="index"
            :style="{ backgroundColor: color }"
            class="color-peg"
          ></span>
        </div>
        <button @click="removeLastColor">Remove Last</button>
        <button @click="submitGuess">Submit Guess</button>
      </div>

      <div v-if="gameOver" class="secret-code">
        <h2>Game Over!</h2>
        <p>The secret code was:</p>
        <div class="secret-pegs">
          <span
            v-for="(color, index) in secretCode"
            :key="index"
            :style="{ backgroundColor: color }"
            class="color-peg secret-peg"
          ></span>
        </div>
      </div>
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
  () => guessCount.value >= 8 || guesses.value.some(isCorrectGuess)
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
  guess: string[] | undefined
): { correct: number; misplaced: number } => {
  if (!guess) return { correct: 0, misplaced: 0 }

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
.mastermind-container {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  min-height: 100vh;
  padding: 20px;
  width: 100%;
}

.game-board {
  width: 40%;
  min-width: 300px;
  max-width: 500px;
  background-color: #f0f0f0;
  border-radius: 10px;
  padding: 20px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  margin: 0 auto;
}

.version-align {
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 14px;
  color: #4169e1;
  font-weight: bold;
}

.board-rows {
  display: flex;
  flex-direction: column-reverse;
  gap: 10px;
  margin-bottom: 20px;
}

.board-row {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 10px;
}

.guess-pegs {
  display: flex;
  gap: 5px;
}

.peg-hole {
  width: 30px;
  height: 30px;
  border-radius: 50%;
  background-color: #ddd;
  display: flex;
  justify-content: center;
  align-items: center;
}

.color-peg {
  width: 26px;
  height: 26px;
  border-radius: 50%;
  border: 2px solid #000;
}

.feedback-pegs {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-left: 10px;
}

.feedback-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  grid-gap: 5px;
  padding: 5px;
  background-color: #ccc;
  border-radius: 5px;
}

.feedback-peg-hole {
  width: 15px;
  height: 15px;
  border-radius: 50%;
  background-color: #ddd;
  display: flex;
  justify-content: center;
  align-items: center;
}

.feedback-peg {
  width: 11px;
  height: 11px;
  border-radius: 50%;
}

.feedback-peg.correct {
  background-color: black;
}

.feedback-peg.misplaced {
  background-color: white;
  border: 1px solid #000;
}

.controls {
  margin-top: 20px;
}

.color-buttons {
  display: flex;
  gap: 10px;
  margin-bottom: 10px;
  justify-content: center;
}

.color-buttons button {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  border: 2px solid #000;
  cursor: pointer;
  padding: 10px;
}

.current-guess {
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 20px 0;
}

.current-guess .color-peg {
  width: 26px;
  height: 26px;
  border-radius: 50%;
  border: 2px solid #000;
  margin-right: 5px;
}

.secret-code {
  margin-top: 20px;
  text-align: center;
}

.secret-pegs {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-top: 10px;
}

.secret-peg {
  width: 30px;
  height: 30px;
  border-radius: 50%;
  border: 2px solid #000;
}

button {
  margin-right: 10px;
  padding: 5px 10px;
  cursor: pointer;
}
</style>
