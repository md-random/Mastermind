# Vue 3 Mastermind Component

## Description

This is a Vue 3 component that implements the classic code-breaking game Mastermind. It features a responsive design, customizable game settings, and an intuitive user interface.

## Features

- Play Mastermind against the computer
- Responsive design for various screen sizes
- Customizable number of attempts (default: 8)
- Color selection for code creation and guessing
- Real-time feedback on guesses
- Game over detection with secret code reveal

## Mastermind Project Version History

### Version 1.0 - 1.5

- **1.0**: Initial release with basic Mastermind gameplay
- **1.1**: Added feedback mechanism for correct and misplaced pegs
- **1.2**: Implemented game over detection and secret code reveal
- **1.3**: Enhanced game board layout and visual feedback
- **1.4**: Refined game board layout, improved user interface, and fixed bugs
- **1.5**: Introduced a 3D wood-like appearance for the play area and adjusted layout for controls and play area positioning

## Logic and Function Explanations

### Core Game Functions

- `startNewGame()`: Initializes a new game, generating a secret code and resetting game state.
- `addColor(color)`: Adds a color to the current guess.
- `removeLastColor()`: Removes the last color from the current guess.
- `submitGuess()`: Submits the current guess for evaluation.
- `getFeedback(guess)`: Evaluates a guess and returns feedback on correct and misplaced pegs.

### Helper Functions

- `generateSecretCode()`: Generates a random secret code.
- `isCorrectGuess(guess)`: Checks if a guess matches the secret code.

### Computed Properties

- `gameOver`: Determines if the game is over based on the number of guesses or a correct guess.

## How to Use

### Customization

You can customize the game by modifying the following:

- Number of attempts (default: 8)
- Colors used in the game
- Code length (default: 4)

## Future Improvements

- Add difficulty levels (e.g., more colors, longer codes)
- Implement a scoring system
- Add a timer for competitive play
- Create a two-player mode

## License

This project is open source and available under the MIT License.
