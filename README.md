# 🎮 Ultimate Tic Tac Toe Pro

A professional web-based Tic Tac Toe game built using **Vue.js**.

🔗 **Live Demo:**  
https://tic-tac-toe-flame-two-72.vercel.app/

---

## 🚀 Features

### 🎯 Game Modes
- 🤖 Player vs AI
- 👥 Player vs Player (Manual Mode)

### 🎯 AI Difficulty Levels
- Easy (Random Move)
- Hard / Impossible (Minimax Algorithm with Alpha-Beta Pruning)

### 🎯 Dynamic Board Sizes
- 3x3
- 4x4
- 5x5

### 🎯 Additional Features
- Undo / Redo
- Theme Switching (Blue, Light, Neon)
- Winner Confetti Animation
- Game Analytics Dashboard
- LocalStorage Data Saving

---

## 🧠 How It Works

1. Player **X** starts the game.
2. After every move, the winner is checked.
3. If in **Player vs AI mode**, the AI automatically makes its move.
4. The game ends when:
   - A player wins  
   - Or the board is full (Draw)

---

## ⚙️ Core Functions

| Function | Purpose |
|----------|----------|
| makeMove() | Handles player move |
| getRandomMove() | AI random move logic |
| getBestMove() | AI optimal move |
| minimax() | AI decision algorithm |
| evaluate() | Checks winner |
| resetGame() | Resets board |
| updateStats() | Updates analytics |

---

## 🛠 Technologies Used

- Vue.js
- JavaScript
- CSS
- Canvas Confetti
- Git & GitHub
- Vercel (Deployment)

---

## 📂 Project Structure
tic-tac-toe/
│
├── public/
├── src/
│ ├── App.vue
│ ├── main.js
│ └── style.css
├── index.html
├── package.json
├── README.md
└── vite.config.js


## 🌍 Deployment

This project is deployed using **Vercel**.

## 👨‍💻 Developer

**Mohd Ubaid**  
B.Tech CSE Student  
Interested in Web Development 