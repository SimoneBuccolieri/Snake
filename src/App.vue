<template>
  <div id="app" class="container mx-auto py-8 flex justify-center flex-col h-screen">
    <div v-if="playBtn">
      <h1 class="bold text-4xl">Snake</h1>
      <button @click="play" class=" text-4xl rounded bg-green-600 text-white py-1 px-4 m-8 hover:bg-green-500">Play</button>
    </div>
    <div>
      <div v-if="!gameOver && !playBtn">
        <h3>Punteggio: {{score}}</h3>
      </div>
      <canvas
          v-if="!gameOver"
          :style="{ display: playBtn ? 'none' : 'block' }"
          ref="canvas"
          @touchstart="startTouch"
          @touchmove.prevent="moveTouch"
          @touchend="endTouch"
      ></canvas>
    </div>
    <div v-if="gameOver" class="game-over">
      <h2 class="bold text-4xl">Game Over</h2>
      <button @click="reset" class="  rounded bg-green-600 text-white px-2 m-8 hover:bg-green-500">Play Again</button>
    </div>
  </div>
</template>


<script>
export default {
  data() {
    return {
      score: 0,
      playBtn: true,
      gameOver: false,
      canvawidth: window.innerWidth * 0.8, // x
      canvaheight: window.innerHeight * 0.8, // y
      snake: {
        x: [100],
        y: [100],
        radius: 8,
        color: 'blue',
        direction: 'right',
        length: 5,
      },
      mela: {
        x: null,
        y: null,
        radius: 10,
        color: 'red',
        presenza: false,
      },
      touchStartX: 0,
      touchStartY: 0,
      touchEndX: 0,
      touchEndY: 0,
    };
  },

  methods: {
    play() {
      this.playBtn = false;
      const canvas = this.$refs.canvas;
      canvas.width = this.canvawidth;
      canvas.height = this.canvaheight;
      const ctx = canvas.getContext('2d');
      window.removeEventListener('keydown', this.changeDirection); // Rimuovi il listener esistente
      window.addEventListener('keydown', this.changeDirection); // Aggiungi il nuovo listener

      this.updateGame(ctx);
    },

    updateGame(ctx) {
      let speed = 100 - this.snake.length * 2; // Aggiusta la velocità in base alla lunghezza del serpente
      if (speed < 50) speed = 50;  // Imposta un limite minimo alla velocità

      this.gameTimer = setInterval(() => {
        switch (this.snake.direction) {
          case "right":
            this.snake.x.unshift(this.snake.x[0] + 5);
            this.snake.y.unshift(this.snake.y[0]);
            break;
          case "left":
            this.snake.x.unshift(this.snake.x[0] - 5);
            this.snake.y.unshift(this.snake.y[0]);
            break;
          case "up":
            this.snake.x.unshift(this.snake.x[0]);
            this.snake.y.unshift(this.snake.y[0] - 5);
            break;
          case "down":
            this.snake.x.unshift(this.snake.x[0]);
            this.snake.y.unshift(this.snake.y[0] + 5);
            break;
        }

        this.resizeSnake();
        this.checkMela();

        ctx.clearRect(0, 0, this.canvawidth, this.canvaheight);
        this.drawGame(ctx);
        this.drawSnake(ctx);
        this.spawnMela(ctx);
        this.checkCollisioni();
        if (this.gameOver) {
          clearInterval(this.gameTimer); // Interrompi il gioco in caso di gameOver
        }
      }, speed);
    },

    checkMela() {
      if (Math.abs(this.snake.x[0] - this.mela.x) < 15 && Math.abs(this.snake.y[0] - this.mela.y) < 15) {
        this.snake.length = this.snake.length + 5;
        this.mela.presenza = false;
        this.score++;
      }
    },

    spawnMela(ctx) {
      if (!this.mela.presenza) {
        this.mela.x = Math.random() * this.canvawidth;
        this.mela.y = Math.random() * this.canvaheight;
        this.mela.presenza = true;
      }
      ctx.beginPath();
      ctx.arc(this.mela.x, this.mela.y, this.mela.radius, 0, Math.PI * 2);
      ctx.fillStyle = this.mela.color;
      ctx.fill();
      ctx.closePath();
    },

    checkCollisioni() {
      // Collisione con i bordi
      if (
          this.snake.x[0] < 0 ||
          this.snake.x[0] > this.canvawidth ||
          this.snake.y[0] < 0 ||
          this.snake.y[0] > this.canvaheight
      ) {
        this.gameOver = true;
        return;
      }

      // Collisione con il corpo
      for (let i = 1; i < this.snake.x.length; i++) {
        if (
            this.snake.x[0] === this.snake.x[i] &&
            this.snake.y[0] === this.snake.y[i]
        ) {
          this.gameOver = true;
          return;
        }
      }
    },

    resizeSnake() {
      this.snake.x = this.snake.x.slice(0, this.snake.length);
      this.snake.y = this.snake.y.slice(0, this.snake.length);
    },

    drawGame(ctx) {
      ctx.fillStyle = 'green';
      ctx.fillRect(0, 0, this.canvawidth, this.canvaheight);
    },

    drawSnake(ctx) {
      for (let i = 0; i < this.snake.x.length; i++) {
        ctx.beginPath();
        ctx.arc(this.snake.x[i], this.snake.y[i], this.snake.radius, 0, Math.PI * 2);
        ctx.fillStyle = this.snake.color;
        ctx.fill();
        ctx.closePath();
      }
    },

    changeDirection(event) {
      const oppositeDirections = {
        up: 'down',
        down: 'up',
        left: 'right',
        right: 'left',
      };

      if (oppositeDirections[this.snake.direction] !== event.key.replace('Arrow', '').toLowerCase()) {
        this.snake.direction = event.key.replace('Arrow', '').toLowerCase();
      }
    },

    reset() {
      this.playBtn = true;
      this.gameOver = false;
      // Reinizializza il serpente
      this.snake = {
        x: [100],
        y: [100],
        radius: 8,
        color: 'red',
        direction: 'right',
        length: 5,
      };

      // Reinizializza la mela
      this.mela = {
        x: null,
        y: null,
        radius: 10,
        color: 'blue',
        presenza: false,
      };
    },

    // Metodi per gestire gli eventi touch
    startTouch(e) {
      this.touchStartX = e.changedTouches[0].screenX;
      this.touchStartY = e.changedTouches[0].screenY;
    },

    moveTouch(e) {
      this.touchEndX = e.changedTouches[0].screenX;
      this.touchEndY = e.changedTouches[0].screenY;
    },

    endTouch() {
      const diffX = this.touchEndX - this.touchStartX;
      const diffY = this.touchEndY - this.touchStartY;

      if (Math.abs(diffX) > Math.abs(diffY)) {
        // Swipe orizzontale
        if (diffX > 0) {
          this.snake.direction = 'right';
        } else {
          this.snake.direction = 'left';
        }
      } else {
        // Swipe verticale
        if (diffY > 0) {
          this.snake.direction = 'down';
        } else {
          this.snake.direction = 'up';
        }
      }
    },
  },
};
</script>


<style scoped>
#app{
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
  background: white;
  overflow: hidden;
}
canvas {
  border: 1px solid black;
}

.game-over {
  text-align: center;
  font-size: 24px;
  margin-top: 20px;
}
</style>
