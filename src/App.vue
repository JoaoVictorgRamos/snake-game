<template>
  <div id="app">
    <div class="score">
      score: <span class="score--value score-roulette">{{ score }}</span>
    </div>
    <div class="menu-screen" :style="{ display: gameover ? 'flex' : 'none' }">
      <span class="game-over">game over</span>
      <span
        >score: <span class="fianl-score">{{ score }}</span></span
      >
      <button class="btn-play" @click="resetGame">
        <span class="material-symbols-outlined"> play_circle </span>
        Jogar novamente
      </button>
    </div>
    <canvas width="600" height="600"></canvas>
  </div>
</template>

<script>
import audioFile from "@/assets/audio.mp3";
export default {
  name: "App",
  components: {},
  data() {
    return {
      size: 30,
      snake: [{ x: 270, y: 240 }],
      initial_postiion: [{ x: 270, y: 240 }],
      direction: null,
      timeInterval: null,
      foodPosition: null,
      score: 0,
      gameover: false,
    };
  },
  methods: {
    resetGame() {
      this.foodPosition = null;
      this.drawFood();
      const canvas = document.querySelector("canvas");
      canvas.style.filter = "none";
      this.score = 0;
      this.gameover = false;
      this.snake = this.initial_postiion;
    },
    gameOver() {
      const canvas = document.querySelector("canvas");
      this.direction = null;
      this.gameover = true;
      canvas.style.filter = "blur(2px)";
    },
    checkCollision() {
      const canvas = document.querySelector("canvas");
      const head = this.snake[this.snake.length - 1];
      const neckIndex = this.snake.length - 2;
      const canvasLimit = canvas.width - this.size;
      const wallColision =
        head.x < 0 ||
        head.x > canvasLimit ||
        head.y < 0 ||
        head.y > canvasLimit;
      const selfCollision = this.snake.find((position, index) => {
        return (
          index < neckIndex && position.x == head.x && position.y == head.y
        );
      });
      if (wallColision || selfCollision) {
        this.gameOver();
      }
    },
    playAudio() {
      const audio = new Audio(audioFile);
      audio.play();
      const scoreElement = document.querySelector(".score-roulette");
      scoreElement.classList.remove("score-roulette");
      void scoreElement.offsetWidth; // Trigger reflow to restart the animation
      scoreElement.classList.add("score-roulette");
    },
    checkEat() {
      const head = this.snake[this.snake.length - 1];
      if (head.x == this.foodPosition.x && head.y == this.foodPosition.y) {
        this.score += 10;
        this.snake.push(head);
        this.playAudio();

        let x = this.randomPosition();
        let y = this.randomPosition();

        while (
          this.snake.find((position) => position.x == x && position.y == y)
        ) {
          x = this.randomPosition();
          y = this.randomPosition();
        }

        this.foodPosition.x = x;
        this.foodPosition.y = y;
        this.foodPosition.color = this.randomColor();
      }
    },
    randomColor() {
      const red = this.randomNumber(0, 255);
      const green = this.randomNumber(0, 255);
      const blue = this.randomNumber(0, 255);
      return `rgb(${red},${green},${blue})`;
    },
    randomNumber(min, max) {
      return Math.round(Math.random() * (max - min) + min);
    },
    randomPosition() {
      const canvas = document.querySelector("canvas");
      const number = this.randomNumber(0, canvas.width - this.size);
      return Math.round(number / 30) * 30;
    },
    drawFood() {
      const canvas = document.querySelector("canvas");
      const ctx = canvas.getContext("2d");

      if (!this.foodPosition) {
        const food = {
          x: this.randomPosition(),
          y: this.randomPosition(),
          color: this.randomColor(),
        };
        const { x, y, color } = food;

        ctx.shadowColor = color;
        ctx.shadowBlur = 6;
        ctx.fillStyle = color;
        ctx.fillRect(x, y, this.size, this.size);
        ctx.shadowBlur = 0;

        this.foodPosition = { x, y, color };
      } else {
        const { x, y, color } = this.foodPosition;

        ctx.shadowColor = color;
        ctx.shadowBlur = 6;
        ctx.fillStyle = color;
        ctx.fillRect(x, y, this.size, this.size);
        ctx.shadowBlur = 0;
      }
    },
    drawGrid() {
      const canvas = document.querySelector("canvas");
      const ctx = canvas.getContext("2d");
      ctx.lineWidth = 1;
      ctx.strokeStyle = "#191919";
      for (let i = 30; i < canvas.width; i += 30) {
        // Desenha as linhas na Vertical -->
        ctx.beginPath();
        ctx.lineTo(i, 0);
        ctx.lineTo(i, 600);
        ctx.stroke();
        // Desenha as linhas na Horizontal -->
        ctx.beginPath();
        ctx.lineTo(0, i);
        ctx.lineTo(600, i);
        ctx.stroke();
      }
    },
    moveSnake() {
      const head = this.snake[this.snake.length - 1];
      if (!this.direction) {
        return;
      }
      if (this.direction == "right") {
        this.snake.push({ x: head.x + this.size, y: head.y });
      }
      if (this.direction == "left") {
        this.snake.push({ x: head.x - this.size, y: head.y });
      }
      if (this.direction == "down") {
        this.snake.push({ x: head.x, y: head.y + this.size });
      }
      if (this.direction == "up") {
        this.snake.push({ x: head.x, y: head.y - this.size });
      }
      this.snake.shift();
    },
    drawSnake() {
      const canvas = document.querySelector("canvas");
      const ctx = canvas.getContext("2d");
      ctx.fillStyle = "#ddd";
      this.snake.forEach((position, index) => {
        if (index == this.snake.length - 1) {
          ctx.fillStyle = "#FFF";
        }
        ctx.fillRect(position.x, position.y, this.size, this.size);
      });
    },
    gameLoop() {
      clearInterval(this.timeInterval);
      const canvas = document.querySelector("canvas");
      const ctx = canvas.getContext("2d");
      ctx.clearRect(0, 0, 600, 600);
      this.drawGrid();
      this.drawFood();
      this.drawSnake();
      this.moveSnake();
      this.checkEat();
      this.checkCollision();
      this.timeInterval = setInterval(() => {
        this.gameLoop();
      }, 300);
    },
  },
  mounted() {
    this.gameLoop();
    document.addEventListener("keydown", ({ key }) => {
      if (this.gameover) {
        return;
      }
      if ((key === "ArrowUp" || key === "w") && this.direction != "down") {
        this.direction = "up";
      }
      if ((key === "ArrowDown" || key === "s") && this.direction != "up") {
        this.direction = "down";
      }
      if ((key === "ArrowRight" || key === "d") && this.direction != "left") {
        this.direction = "right";
      }
      if ((key === "ArrowLeft" || key === "a") && this.direction != "right") {
        this.direction = "left";
      }
    });
  },
};
</script>

<style lang="scss">
#app {
  position: relative;
}
* {
  margin: 0;
  padding: 0px;
  box-sizing: border-box;
  font-family: "Poppings", sans-serif;
}

body {
  background: #191919;
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  align-items: center;
  justify-content: center;
  color: #fff;
  text-align: center;
}

canvas {
  background: #111;
  position: relative;
}

.score {
  display: flex;
  flex-direction: column;
  align-items: center;
  font-size: 1.8rem;
}

.score--value {
  font-weight: 700;
  font-size: 4.5rem;
  display: block;
  margin-top: -10px;
}

.menu-screen {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  display: none;
  flex-direction: column;
  align-items: center;
  z-index: 1;
}

.game-over {
  text-transform: uppercase;
  font-weight: 700;
  font-size: 3rem;
}

.final-score {
  font-weight: 500;
  font-size: 1.5rem;
}

.btn-play {
  border: none;
  border-radius: 100px;
  padding: 10px 15px 10px 12px;
  font-size: 1rem;
  font-weight: 600;
  color: #333;
  margin-top: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 5px;
  cursor: pointer;
}
.score-roulette {
  animation: scoreAnimation 0.3s;
}

@keyframes scoreAnimation {
  0% {
    transform: rotateX(0);
  }
  100% {
    transform: rotateX(360deg);
  }
}
</style>
