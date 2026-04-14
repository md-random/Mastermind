<template>
  <div class="mastermind-container">
    <audio ref="audioPlayer1" :src="audioSrc1" @timeupdate="onAudioTimeUpdate(1)" @ended="onAudioEnded(1)" class="bg-audio"></audio>
    <audio ref="audioPlayer2" :src="audioSrc2" @timeupdate="onAudioTimeUpdate(2)" @ended="onAudioEnded(2)" class="bg-audio"></audio>
    
    <div class="controls">
      <div class="header-container">
        <div class="audio-controls">
          <button @click="toggleGameMute" class="icon-btn">
            {{ isMuted ? '🔇' : '🔊' }}
          </button>
          <input 
            v-if="!isMuted"
            type="range" 
            min="0" 
            max="1" 
            step="0.05" 
            v-model.number="volume" 
            @input="onVolumeChange"
            class="volume-slider"
          />
        </div>
        <div v-if="!gameOver" class="title-align">
          <h1>Mastermind <span class="version-align">version 2.5</span></h1>
        </div>
      </div>
      <div v-if="!gameOver">
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
      </div>
      <div v-else class="secret-code">
        <h1>Game Over!</h1>
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
      <div class="action-buttons">
        <button @click="removeLastColor" :disabled="gameOver || currentGuess.length === 0">
          Remove Last
        </button>
        <button @click="submitGuess" :disabled="gameOver || currentGuess.length < codeLength">Submit Guess</button>
        <button @click="startNewGame">New Game</button>
      </div>
    </div>
    <div class="play-area">
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
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted, nextTick } from 'vue'

const audioPlayer1 = ref<HTMLAudioElement | null>(null)
const audioPlayer2 = ref<HTMLAudioElement | null>(null)
const playlist = [
  '/Titan_s_Ascent.mp3',
  '/Sol_Radiante (1).mp3',
  '/Echoes_in_the_Lavender_Haze.mp3'
]
const trackIndex = ref(0)
const audioSrc1 = ref(playlist[0])
const audioSrc2 = ref('')
const isMuted = ref(false)
const volume = ref(0.35)

let isCrossfading = false
let activeAudioInterval: ReturnType<typeof setInterval> | null = null

const crossfadeTracks = (sourceEl: HTMLMediaElement | null, targetEl: HTMLMediaElement) => {
  if (!targetEl || isMuted.value) return
  isCrossfading = true
  
  const targetVol = volume.value
  const startSourceVol = sourceEl ? sourceEl.volume : 0
  
  targetEl.volume = 0
  targetEl.muted = false
  const playPromise = targetEl.play()
  if (playPromise) playPromise.catch(e => console.warn('Autoplay blocked', e))
  
  let progress = 0
  const steps = 30
  const stepTime = 3000 / steps
  
  if (activeAudioInterval) clearInterval(activeAudioInterval)
  activeAudioInterval = setInterval(() => {
    progress++
    const ratio = progress / steps
    
    if (!isMuted.value) {
      if (sourceEl) sourceEl.volume = Math.max(0, startSourceVol * (1 - ratio))
      targetEl.volume = Math.min(targetVol, targetVol * ratio)
    }
    
    if (progress >= steps) {
      if (activeAudioInterval) clearInterval(activeAudioInterval)
      isCrossfading = false
      if (sourceEl) {
        sourceEl.pause()
        sourceEl.currentTime = 0
      }
    }
  }, stepTime)
}

const onAudioTimeUpdate = (playerNum: number) => {
  if (isMuted.value || isCrossfading) return
  const currentAudio = playerNum === 1 ? audioPlayer1.value : audioPlayer2.value
  if (!currentAudio || isNaN(currentAudio.duration)) return
  
  const timeLeft = currentAudio.duration - currentAudio.currentTime
  if (timeLeft <= 3.0 && timeLeft > 0) {
    const nextAudio = playerNum === 1 ? audioPlayer2.value : audioPlayer1.value
    
    trackIndex.value = (trackIndex.value + 1) % playlist.length
    if (playerNum === 1) {
      audioSrc2.value = playlist[trackIndex.value]
    } else {
      audioSrc1.value = playlist[trackIndex.value]
    }
    
    nextTick(() => {
      setTimeout(() => {
        if (nextAudio) crossfadeTracks(currentAudio as HTMLMediaElement, nextAudio as HTMLMediaElement)
      }, 100)
    })
  }
}

const onAudioEnded = (playerNum: number) => {
  if (!isCrossfading) onAudioTimeUpdate(playerNum)
}

const onVolumeChange = () => {
  const activePlayer = (audioPlayer1.value && !audioPlayer1.value.paused) ? audioPlayer1.value :
                       (audioPlayer2.value && !audioPlayer2.value.paused) ? audioPlayer2.value : null
  
  if (activePlayer && !isCrossfading) {
    activePlayer.volume = volume.value
  }
  
  if (volume.value === 0) {
    isMuted.value = true
  } else if (isMuted.value) {
    isMuted.value = false
  }
}

const toggleGameMute = () => {
  isMuted.value = !isMuted.value
  const activePlayer = (audioPlayer1.value && !audioPlayer1.value.paused) ? audioPlayer1.value :
                       (audioPlayer2.value && !audioPlayer2.value.paused) ? audioPlayer2.value : null
  
  if (isMuted.value) {
    if (activePlayer) {
      activePlayer.muted = true
      activePlayer.pause()
    }
    if (activeAudioInterval) clearInterval(activeAudioInterval)
    isCrossfading = false
  } else {
    if (activePlayer) {
      activePlayer.muted = false
      activePlayer.volume = volume.value
      activePlayer.play().catch(() => {})
    } else if (audioPlayer1.value) {
      crossfadeTracks(null, audioPlayer1.value as HTMLMediaElement)
    }
  }
}

onMounted(() => {
  if (audioPlayer1.value) {
    crossfadeTracks(null, audioPlayer1.value as HTMLMediaElement)
  }
})

onUnmounted(() => {
  if (activeAudioInterval) clearInterval(activeAudioInterval)
})

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

startNewGame()
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Playfair+Display:ital,wght@0,400..900;1,400..900&display=swap');
.mastermind-container {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 5%;
  min-height: 100vh;
  padding: 20px;
  width: 100%;
  box-sizing: border-box;
  font-family: 'Playfair Display', serif;
  position: relative;
  z-index: 1;
}

.mastermind-container::before {
  content: '';
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  background: url('../assets/CherryBlossomMastermind.png') center/cover no-repeat;
  filter: blur(12px) brightness(0.4);
  transform: scale(1.1);
  z-index: -1;
  pointer-events: none;
}

.controls {
  width: 45%;
  max-width: 500px;
  min-width: 320px;
  color: #fff; 
  text-shadow: 0 4px 10px rgba(0, 0, 0, 0.8);
}

.header-container {
  display: flex;
  flex-direction: column; /* Center precisely above the title */
  align-items: center;
  justify-content: center;
  gap: 10px;
  margin-bottom: 20px;
}

.audio-controls {
  position: relative; /* Fixed anchor for the drop-down slider */
  width: 32px;
  height: 32px;
  margin-bottom: 90px; /* Pushes it 20px higher above the Mastermind title */
  z-index: 100;
}

.icon-btn {
  position: absolute;
  top: 0;
  left: 0;
  background-color: rgba(0, 0, 0, 0.4) !important;
  box-shadow: 0 0 10px rgba(0,0,0,0.5) !important;
  color: #fff;
  border: 1px solid rgba(255, 255, 255, 0.3) !important;
  border-radius: 50%;
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.96rem;
  transition: all 0.2s;
  cursor: pointer;
}

.icon-btn:hover {
  background-color: rgba(255, 255, 255, 0.1) !important;
  transform: scale(1.1);
}

.volume-slider {
  position: absolute;
  top: 82px; /* Positions perfectly below the icon */
  left: 16px; /* Perfectly aligned to the horizontal center of the 32px icon */
  width: 16px; 
  height: 100px; /* The vertical track length bounds */
  transform: translate(-50%, -50%) rotate(180deg); /* Reversed direction: bottom is louder */
  -webkit-appearance: slider-vertical; /* Flawless native vertical finger swiping */
  touch-action: none; /* Stops screen from sliding */
  cursor: pointer;
  margin: 0;
}

.version-align {
  display: flex;
  align-items: center;
  font-size: small;
  color: #ffb7c5;
  padding-left: 20px;
}

h1 {
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 3em;
  color: #fff;
  margin: 0;
}

.play-area {
  width: 35%;
  max-width: 400px;
  min-width: 300px;
  display: flex;
  justify-content: center;
}

.game-board {
  width: 100%;
  border-radius: 10px;
  padding: 20px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.board-rows {
  display: flex;
  flex-direction: column-reverse;
  gap: 15px;
  margin-bottom: 20px;
  border-radius: 10px;
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.19), 0 6px 6px rgba(0, 0, 0, 0.23),
    inset 0 0 50px rgba(0, 0, 0, 0.5);
  padding: 30px 46px;
  width: max-content;
  background-color: #5296a5;
}

.board-row {
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: transparent;
  border-radius: 5px;
  padding: 18px 10px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1),
    inset 0 1px 3px rgba(255, 255, 255, 0.2);
}

.guess-pegs {
  display: flex;
  gap: 5px;
}

.peg-hole {
  width: 30px;
  height: 30px;
  border-radius: 50%;
  background-color: #140f0989;
  display: flex;
  justify-content: center;
  align-items: center;
  box-shadow: inset 0 0 10px rgba(0, 0, 0, 0.5);
}

.color-peg {
  width: 26px;
  height: 26px;
  border-radius: 50%;
  border: 2px solid #000;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
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
  background-color: #cccccc49;
  border-radius: 5px;
}

.feedback-peg-hole {
  width: 15px;
  height: 15px;
  border-radius: 50%;
  background-color: #1f0c0c33;
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

.color-buttons {
  display: flex;
  gap: 10px;
  justify-content: center;
}

.color-buttons button {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  border: 2px solid #000;
  cursor: pointer;
  padding: 10px;
  transition: transform 0.1s cubic-bezier(0.2, 0.8, 0.2, 1);
}

.color-buttons button:active:not(:disabled) {
  transform: scale(0.75);
}

.current-guess {
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 20px 0;
  min-height: 40px; /* Pre-reserves the space for the pegs so it never jumps */
  color: #fff;
  font-weight: bold;
  letter-spacing: 1px;
}

.current-guess .color-peg {
  width: 26px;
  height: 26px;
  border-radius: 50%;
  border: 2px solid #000;
  margin: 5px;
}

.secret-code {
  margin-top: 20px;
  text-align: center;
  color: #fff;
  font-weight: bold;
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
  background-color: #5296a5;
  color: #fcd7ad;
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.19), 0 6px 6px rgba(0, 0, 0, 0.23),
    inset 0 0 50px rgba(0, 0, 0, 0.5);
  border: none !important;
  transition: opacity 0.2s, transform 0.1s;
}

button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.action-buttons {
  display: flex;
  justify-content: center;
  gap: 10px;
  width: 100%;
  margin-top: 10px;
}

.action-buttons button {
  border-radius: 15px;
  padding: 8px 16px;
}

@media (max-width: 768px) {
  .mastermind-container {
    flex-direction: column;
    justify-content: flex-start;
    padding: 10px;
    min-height: 100vh;
    height: auto;
    overflow: visible;
  }

  .controls {
    width: 100%;
    position: static;
    transform: none;
    order: 2;
    display: flex;
    flex-direction: column;
    align-items: center;
    height: inherit;
  }

  .header-container {
    flex-direction: row; /* Enforce side-by-side horizontally even on small screens */
    gap: 10px;
    width: 100%;
    justify-content: center;
  }
  
  .audio-controls {
    position: fixed;
    left: 15px;
    top: 25%; /* Moved up to roughly 25% of the screen height */
    margin-top: -16px; /* Locks vertical alignment down exactly so adding the slider height doesn't force it to jump */
  }
  
  .play-area {
    width: 300px;
    position: static;
    transform: none;
    order: 1;
    margin: 0 auto;
  }

  h1 {
    font-size: 1.5rem;
    margin: 0 !important;
    padding-bottom: 20px;
  }

  .board-rows {
    transform: none;
    border-width: 10px;
    padding: 10px;
  }

  .board-row {
    padding: 5px;
  }

  .peg-hole,
  .color-peg {
    width: 20px;
    height: 20px;
  }

  .feedback-peg-hole {
    width: 10px;
    height: 10px;
  }

  .feedback-peg {
    width: 8px;
    height: 8px;
  }

  .color-buttons button {
    width: 30px;
    height: 30px;
    padding: 5px;
  }

  .current-guess .color-peg {
    width: 20px;
    height: 20px;
  }

  .secret-peg {
    width: 20px;
    height: 20px;
  }

  .current-guess {
    margin: 10px 20px;
    min-height: 35px;
  }

  button {
    padding: 3px 6px;
    font-size: 0.7rem;
  }
}
</style>
