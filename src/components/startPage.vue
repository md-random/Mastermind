<template>
  <div class="start-page-container">
    <div class="card">
      <h1 class="title">Mastermind</h1>
      
      <div class="instructions">
        <h2>How to Play</h2>
        <p>
          Mastermind is a code-breaking game. Your goal is to guess the secret code of colored pegs within a certain number of turns.
        </p>
        
        <div class="feedback-rules">
          <h3>Feedback Pegs:</h3>
          <ul>
            <li>
              <span class="peg black"></span>
              <div>
                <strong>Black Peg:</strong> A peg is the correct color and in the correct position.
              </div>
            </li>
            <li>
              <span class="peg white"></span>
              <div>
                <strong>White Peg:</strong> A peg is the correct color but in the wrong position.
              </div>
            </li>
            <li>
              <div class="no-peg-indent">
                <strong>No Peg:</strong> The guessed color is not in the secret code.
              </div>
            </li>
          </ul>
        </div>
        
        <div class="game-controls">
          <h3>Game Controls:</h3>
          <ul>
             <li>
               <div class="demo-colors">
                 <span class="color-peg" style="background-color: red;"></span>
                 <span class="color-peg" style="background-color: blue;"></span>
                 <span class="color-peg" style="background-color: green;"></span>
                 <span class="color-peg" style="background-color: yellow;"></span>
                 <span class="color-peg" style="background-color: purple;"></span>
                 <span class="color-peg" style="background-color: orange;"></span>
               </div>
               <div><strong>Select Colors:</strong> Click these pegs to add them to your guess.</div>
             </li>
             <li>
               <button class="demo-btn">Remove Last</button>
               <div><strong>Undo:</strong> Click to remove the last color if you made a mistake.</div>
             </li>
             <li>
               <button class="demo-btn">Submit Guess</button>
               <div><strong>Submit:</strong> Click when you have selected 4 colors to see your feedback.</div>
             </li>
          </ul>
        </div>

        <p class="hint">Use the feedback to deduce the secret code. Good luck!</p>
      </div>

      <button class="start-btn" @click="$emit('start')">Play Game</button>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  name: 'startPage',
  emits: ['start'],
})
</script>

<style scoped>
.start-page-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  padding: 1rem;
}

.card {
  background: rgba(255, 255, 255, 0.85);
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  border-radius: 16px;
  padding: 2.5rem;
  max-width: 600px;
  width: 100%;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
  text-align: left;
  border: 1px solid rgba(255, 255, 255, 0.3);
}

.title {
  text-align: center;
  font-size: 3rem;
  margin-top: 0;
  margin-bottom: 1.5rem;
  color: #2c3e50;
  text-transform: uppercase;
  letter-spacing: 2px;
}

.instructions {
  color: #34495e;
  line-height: 1.6;
  font-size: 1.1rem;
}

.instructions h2 {
  font-size: 1.5rem;
  margin-bottom: 0.5rem;
  color: #2c3e50;
  border-bottom: 2px solid #e2e8f0;
  padding-bottom: 0.5rem;
}

.feedback-rules {
  margin-top: 1.5rem;
  background: rgba(255, 255, 255, 0.5);
  padding: 1rem;
  border-radius: 8px;
}

.feedback-rules h3 {
  margin-top: 0;
  font-size: 1.25rem;
  color: #2c3e50;
}

.feedback-rules ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

.feedback-rules li {
  display: flex;
  align-items: flex-start;
  margin-bottom: 0.75rem;
}

.peg {
  display: inline-block;
  flex-shrink: 0;
  margin-top: 3px;
  width: 16px;
  height: 16px;
  border-radius: 50%;
  margin-right: 12px;
  box-shadow: inset 0 2px 4px rgba(0,0,0,0.3);
  border: 1px solid rgba(0,0,0,0.1);
}

.no-peg-indent {
  margin-left: 28px;
}

.game-controls {
  margin-top: 1.5rem;
  background: rgba(255, 255, 255, 0.5);
  padding: 1rem;
  border-radius: 8px;
}

.game-controls h3 {
  margin-top: 0;
  font-size: 1.25rem;
  color: #2c3e50;
  margin-bottom: 1rem;
}

.game-controls ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

.game-controls li {
  display: flex;
  align-items: center;
  margin-bottom: 1.25rem;
  gap: 15px;
}

.game-controls li:last-child {
  margin-bottom: 0;
}

.demo-colors {
  display: flex;
  gap: 6px;
  flex-shrink: 0;
}

.color-peg {
  width: 22px;
  height: 22px;
  border-radius: 50%;
  border: 2px solid #000;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  display: inline-block;
}

.demo-btn {
  padding: 6px 12px;
  background-color: #5296a5;
  color: #fcd7ad;
  border-radius: 12px;
  border: none;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.19);
  font-size: 0.85rem;
  font-weight: 600;
  flex-shrink: 0;
  cursor: default;
  white-space: nowrap;
}

@media (max-width: 768px) {
  .title {
    font-size: 1.8rem;
    letter-spacing: 1px;
    word-wrap: break-word;
    margin-bottom: 1rem;
  }
  .card {
    padding: 1.5rem 1rem;
    margin: 0;
  }
  .instructions {
    font-size: 0.95rem;
  }
  .instructions h2 {
    font-size: 1.3rem;
  }
  .feedback-rules h3 {
    font-size: 1.1rem;
  }
  .no-peg-indent {
    margin-left: 28px;
  }
  .game-controls li {
    flex-direction: column;
    align-items: flex-start;
    gap: 8px;
  }
}

.peg.black {
  background-color: #222;
}

.peg.white {
  background-color: #f8f9fa;
}

.hint {
  font-style: italic;
  margin-top: 1.5rem;
  text-align: center;
  color: #7f8c8d;
}

.start-btn {
  display: block;
  width: 100%;
  padding: 1rem;
  margin-top: 2rem;
  font-size: 1.25rem;
  font-weight: bold;
  color: white;
  background-color: #3498db;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 6px rgba(52, 152, 219, 0.3);
}

.start-btn:hover {
  background-color: #2980b9;
  transform: translateY(-2px);
  box-shadow: 0 6px 12px rgba(52, 152, 219, 0.4);
}

.start-btn:active {
  transform: translateY(0);
  box-shadow: 0 2px 4px rgba(52, 152, 219, 0.4);
}
</style>
