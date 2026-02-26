<template>
  <div :class="['container', theme]">
    <h1>Ultimate Tic Tac Toe Pro</h1>

    <!-- MODE SELECTION -->
    <div class="mode-selection">
      <button 
        :class="{active: gameMode==='ai'}"
        @click="setMode('ai')">
        🤖 Player vs AI
      </button>

      <button 
        :class="{active: gameMode==='two'}"
        @click="setMode('two')">
        👥 Player vs Player
      </button>
    </div>

    <!-- SETTINGS -->
    <div class="settings">

      <!-- Difficulty only for AI -->
      <select v-if="gameMode==='ai'" v-model="difficulty">
        <option value="easy">Easy</option>
        <option value="medium">Medium</option>
        <option value="hard">Hard</option>
        <option value="impossible">Impossible</option>
      </select>

      <select v-model="size" @change="resetGame">
        <option :value="3">3x3</option>
        <option :value="4">4x4</option>
        <option :value="5">5x5</option>
      </select>

      <button @click="toggleTheme">Switch Theme</button>
    </div>

    <p v-if="aiThinking">AI is thinking...</p>

    <!-- BOARD -->
    <div class="board" :style="{gridTemplateColumns:`repeat(${size},1fr)`}">
      <div
        v-for="(cell,index) in board"
        :key="index"
        class="cell"
        @click="makeMove(index)"
      >
        {{ cell }}
      </div>
    </div>

    <h2>{{ status }}</h2>

    <!-- CONTROLS -->
    <div class="controls">
      <button @click="undoMove">Undo</button>
      <button @click="redoMove">Redo</button>
      <button @click="resetGame">Restart</button>
    </div>

    <!-- DASHBOARD -->
    <div class="dashboard">
      <h3>Analytics</h3>
      <p>Total Games: {{ stats.total }}</p>
      <p>X Wins: {{ stats.xWins }}</p>
      <p>O Wins: {{ stats.oWins }}</p>
      <p>Draws: {{ stats.draws }}</p>
    </div>
  </div>
</template>

<script>
import confetti from "canvas-confetti";

export default {
  data(){
    return{
      size:3,
      board:[],
      history:[],
      redoStack:[],
      currentPlayer:"X",
      winner:null,
      difficulty:"hard",
      aiThinking:false,
      gameMode:"ai",   // 🔥 NEW
      theme: localStorage.getItem("theme") || "blue",
      stats: JSON.parse(localStorage.getItem("stats")) || {
        total:0,xWins:0,oWins:0,draws:0
      }
    }
  },

  computed:{
    status(){
      if(this.winner) return `Winner: ${this.winner}`;
      return `Turn: ${this.currentPlayer}`;
    }
  },

  mounted(){
    this.resetGame();
  },

  methods:{

    setMode(mode){
      this.gameMode = mode;
      this.resetGame();
    },

    toggleTheme(){
      if(this.theme==="blue") this.theme="neon";
      else if(this.theme==="neon") this.theme="light";
      else this.theme="blue";

      localStorage.setItem("theme",this.theme);
    },

    resetGame(){
      this.board=Array(this.size*this.size).fill("");
      this.history=[];
      this.redoStack=[];
      this.currentPlayer="X";
      this.winner=null;
    },

    makeMove(index){
      if(this.board[index] || this.winner) return;

      this.history.push([...this.board]);
      this.board[index]=this.currentPlayer;
      this.redoStack=[];
      this.checkWinner();

      this.currentPlayer=this.currentPlayer==="X"?"O":"X";

      // 🔥 AI Only If AI Mode
      if(
        this.gameMode==="ai" &&
        this.currentPlayer==="O" &&
        !this.winner
      ){
        this.aiThinking=true;

        setTimeout(()=>{
          let move;

          if(this.difficulty==="easy")
            move=this.getRandomMove();
          else
            move=this.getBestMove();

          if(move!==undefined)
            this.makeMove(move);

          this.aiThinking=false;
        },800);
      }
    },

    undoMove(){
      if(!this.history.length) return;
      this.redoStack.push([...this.board]);
      this.board=this.history.pop();
      this.winner=null;
    },

    redoMove(){
      if(!this.redoStack.length) return;
      this.history.push([...this.board]);
      this.board=this.redoStack.pop();
    },

    getRandomMove(){
      const empty=this.board
        .map((v,i)=>v===""?i:null)
        .filter(v=>v!==null);
      return empty[Math.floor(Math.random()*empty.length)];
    },

    getBestMove(){
      let best=-Infinity,move;
      for(let i=0;i<this.board.length;i++){
        if(this.board[i]===""){
          this.board[i]="O";
          let score=this.minimax(this.board,0,false,-Infinity,Infinity);
          this.board[i]="";
          if(score>best){best=score;move=i;}
        }
      }
      return move;
    },

    minimax(board,depth,isMax,alpha,beta){
      const score=this.evaluate(board);
      if(score!==null) return score-depth;

      if(isMax){
        let best=-Infinity;
        for(let i=0;i<board.length;i++){
          if(board[i]===""){
            board[i]="O";
            best=Math.max(best,this.minimax(board,depth+1,false,alpha,beta));
            board[i]="";
            alpha=Math.max(alpha,best);
            if(beta<=alpha) break;
          }
        }
        return best;
      }else{
        let best=Infinity;
        for(let i=0;i<board.length;i++){
          if(board[i]===""){
            board[i]="X";
            best=Math.min(best,this.minimax(board,depth+1,true,alpha,beta));
            board[i]="";
            beta=Math.min(beta,best);
            if(beta<=alpha) break;
          }
        }
        return best;
      }
    },

    evaluate(board){
      const size=this.size;
      const lines=[];
      for(let i=0;i<size;i++){
        lines.push([...Array(size).keys()].map(j=>i*size+j));
        lines.push([...Array(size).keys()].map(j=>j*size+i));
      }
      lines.push([...Array(size).keys()].map(i=>i*size+i));
      lines.push([...Array(size).keys()].map(i=>i*size+(size-1-i)));

      for(let line of lines){
        const values=line.map(i=>board[i]);
        if(values.every(v=>v==="O")) return 10;
        if(values.every(v=>v==="X")) return -10;
      }

      if(!board.includes("")) return 0;
      return null;
    },

    checkWinner(){
      const result=this.evaluate(this.board);
      if(result===10){
        this.winner="O";
        this.updateStats("O");
        confetti({particleCount:120});
      }else if(result===-10){
        this.winner="X";
        this.updateStats("X");
        confetti({particleCount:120});
      }else if(result===0){
        this.winner="Draw";
        this.updateStats("Draw");
      }
    },

    updateStats(winner){
      this.stats.total++;
      if(winner==="X") this.stats.xWins++;
      if(winner==="O") this.stats.oWins++;
      if(winner==="Draw") this.stats.draws++;
      localStorage.setItem("stats",JSON.stringify(this.stats));
    }
  }
}
</script>

<style>
.container{
  text-align:center;
  padding:20px;
  min-height:100vh;
  transition:0.3s;
}

.mode-selection{
  margin:15px 0;
}

.mode-selection button{
  margin:5px;
  padding:8px 15px;
  border:none;
  border-radius:6px;
  cursor:pointer;
  font-weight:bold;
}

.mode-selection button.active{
  background:#2563eb;
  color:white;
}

.blue{ background:#f4f9ff; color:#1e293b; }
.blue .cell{ background:#dbeafe; border:2px solid #93c5fd; }

.light{ background:#ffffff; color:#222; }
.light .cell{ background:#e5e7eb; border:2px solid #cbd5e1; }

.neon{ background:#0f172a; color:#00ffff; }
.neon .cell{
  background:#111827;
  border:2px solid #00ffff;
  box-shadow:0 0 10px #00ffff;
}

.board{
  display:grid;
  gap:10px;
  max-width:400px;
  margin:20px auto;
}

.cell{
  aspect-ratio:1;
  border-radius:10px;
  display:flex;
  align-items:center;
  justify-content:center;
  font-size:1.5rem;
  cursor:pointer;
  transition:0.3s;
}

.controls button{ margin:5px; }
.dashboard{ margin-top:20px; }
</style>