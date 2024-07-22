# Vue 3 Mastermind Game

A simple implementation of the Mastermind game using Vue 3 Composition API with `<script setup>` and TypeScript.

## Features

- Random secret code generation
- Color selection for guesses
- Feedback on correct and misplaced colors
- Game over detection

## Usage

Import the Mastermind component into your Vue 3 application:

```vue
<template>
  <Mastermind />
</template>

<script setup>
import Mastermind from './Mastermind.vue'
</script>
```

## Summary of Changes in Version 1.0

### Game Logic

- **Random Secret Code Generation:** The game generates a random secret code from a predefined set of colors at the start of each game.
- **Color Selection:** Players can select colors for their guesses using buttons.
- **Feedback:** After each guess, the game provides feedback on the number of correct and misplaced colors.
- **Game Over Detection:** The game ends after 10 guesses or when the correct code is guessed.

### New Features

- **New Game Button:** Added a "New Game" button that allows players to start a new game at any time, resetting the game state.
- **Guess Counter:** Added a guess counter that displays the number of guesses made out of the maximum allowed (10).

## License

MIT

### How to Use the Mastermind Component

1. **Import the Component:**

   - Import the `Mastermind.vue` component into your Vue 3 application.

2. **Include in Template:**

   - Use the `<Mastermind />` tag in your template to render the game.

3. **Run the Application:**
   - Start your Vue development server to see the game in action.

### Additional Notes

- The game logic and state are managed using Vue 3's Composition API and `<script setup>`.
- The game provides immediate feedback on each guess, helping players deduce the secret code.
- The "New Game" button resets the game state, allowing for continuous play without refreshing the page.

This `README.md` provides a clear overview of the features and usage of the Mastermind game component, along with a summary of the changes made for version 1.0.

version 1.0.

- Add core game logic with random code generation
- Implement color selection and guess submission
- Add feedback system for correct and misplaced colors
- Implement game over detection
- Add New Game button for resetting game state
- Include guess counter (max 10 guesses)
- Create README with feature summary and usage instructions
