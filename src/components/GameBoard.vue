<template>
  <div class="container">
    <div class="controls">
      <button
        class="btn-primary"
        :class="{ 'btn-disabled': isStarted }"
        @click="startGame"
      >
        {{ isPaused ? 'Restart' : 'Start' }}
      </button>
      <button class="btn-info" @click="pauseGame">
        {{ isPaused ? 'Resume' : 'Pause' }}
      </button>
      <button
        class="btn-danger"
        :class="{ 'btn-disabled': !isStarted }"
        @click="stopGame"
      >
        Stop
      </button>
    </div>
    <div class="speed-control">
      <span class="speed-label">{{ score }}</span>
      <!-- <input
        class="speed-input"
        type="range"
        :min="0.01"
        :max="1"
        :step="0.01"
        v-model="speed"
      /> -->
    </div>
    <div class="baord">
      <div class="board-row" v-for="(row, index) in grid" :key="index">
        <div :class="snakeClassObject(col)" v-for="col in row" :key="col">
          <span v-if="fruit === col" class="fruit"> </span>
          <span v-if="col === snake.slice(-1).pop()" class="snake-eye"> </span>
          <span v-if="col === snake.slice(-1).pop()" class="snake-eye"> </span>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  data: () => ({
    width: 15,
    height: 15,
    grid: null,
    fruit: -1,
    snake: [107, 108, 109],
    score: 0,
    directions: 'ArrowRight',
    prevDirection: 'ArrowUp',
    speed: 0.2,
    interval: null,
    isStarted: false,
    isPaused: false,
    prohibited: {
      ArrowRightArrowLeft: true,
      ArrowLeftArrowRight: true,
      ArrowUpArrowDown: true,
      ArrowDownArrowUp: true,
    },
    snakeHeadClasses: {
      ArrowUp: 'snake-head-up snake-eye-bt',
      ArrowDown: 'snake-head-down snake-eye-bt',
      ArrowRight: 'snake-head-right snake-eye-rl',
      ArrowLeft: 'snake-head-left snake-eye-rl',
    },
  }),

  created() {
    this.setUpGame();
    this.addControls();
  },

  watch: {
    directions() {
      clearInterval(this.interval);
      this.handleKeyUp();
      this.startGame();
    },

    speed() {
      clearInterval(this.interval);
      this.startGame();
    },
  },

  methods: {
    snakeClassObject(col) {
      return {
        'board-col': true,
        snake: this.snake.includes(col),
        'snake-head': col === this.snake.slice(-1).pop(),
        [this.snakeHeadClasses[this.directions]]:
          col === this.snake.slice(-1).pop(),
      };
    },

    setUpGame() {
      const h = this.height;
      const w = this.width;
      const temp = [];

      for (let r = 0; r < w; r++) {
        const row = [];
        for (let c = 0; c < h; c++) {
          const item = this.height * r + c + 1;
          row.push(item);
        }
        temp.push(row);
      }
      this.grid = temp;
      this.setupFruit();
    },

    setupFruit() {
      const tempList = this.grid.flat().filter((e) => !this.snake.includes(e));
      this.fruit = tempList[Math.floor(Math.random() * tempList.length)];
    },

    addControls() {
      const keys = ['ArrowUp', 'ArrowDown', 'ArrowLeft', 'ArrowRight'];
      window.addEventListener('keydown', (e) => {
        if (this.isRightKey(keys, e)) {
          if (e.key === this.directions) return;
          this.prevDirection = this.directions;
          this.directions = e.key;
        }
        if (e.key === 'Enter') this.startGame();
        if (e.key === 'Escape') this.stopGame();
      });
    },

    startGame() {
      this.isStarted = true;
      this.isPaused = false;
      this.interval = setInterval(this.handleKeyUp, this.speed * 1000);
    },

    handleKeyUp() {
      if (this.directions === 'ArrowRight') this.moveRight();
      else if (this.directions === 'ArrowDown') this.moveDown();
      else if (this.directions === 'ArrowLeft') this.moveLeft();
      else if (this.directions === 'ArrowUp') this.moveUp();
    },

    moveRight() {
      const last = this.getLast();
      const temp = this.addFruit(last);
      this.snake = [...temp, last];
    },

    moveLeft() {
      const last = this.getLast();
      const temp = this.addFruit(last);
      this.snake = [...temp, last];
    },

    moveUp() {
      const last = this.snake.slice(-1).pop() - this.height;
      const temp = this.addFruit(last);
      this.snake = [...temp, last];
    },

    moveDown() {
      const last = this.snake.slice(-1).pop() + this.height;
      const temp = this.addFruit(last);
      this.snake = [...temp, last];
    },

    addFruit(last) {
      let temp;
      if (last === this.fruit) {
        this.score++;
        this.setupFruit();
        return this.snake;
      } else {
        temp = this.snake.slice(1);
      }
      return temp;
    },

    getLast() {
      if (this.directions === 'ArrowRight') {
        if (['ArrowUp', 'ArrowDown'].includes(this.prevDirection))
          return this.snake.slice(-1).pop() + 1;
      } else if (this.directions === 'ArrowLeft') {
        if (['ArrowUp', 'ArrowDown'].includes(this.prevDirection))
          return this.snake.slice(-1).pop() - 1;
      }
    },

    stopGame() {
      clearInterval(this.interval);
      this.resetState();
    },

    pauseGame() {
      if (this.isPaused) {
        this.startGame();
        return;
      }
      clearInterval(this.interval);
      this.isPaused = true;
      this.isStarted = false;
    },

    resetState() {
      this.directions = 'ArrowRight';
      this.prevDirection = 'ArrowUp';
      this.isStarted = false;
      this.isPaused = false;
      this.snake = [107, 108, 109];
      this.speed = 0.2;
      this.score = 0;
    },

    isRightKey(keys, e) {
      return (
        keys.includes(e.key) && !this.prohibited[`${e.key}${this.directions}`]
      );
    },
  },

  beforeDestroy() {
    this.resetState();
  },
};
</script>
<style scoped>
.container {
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background: rgb(255, 255, 255);
}

.controls {
  margin: 1em 0;
}
.speed-control {
  margin-bottom: 1em;
  width: 30%;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.speed-input {
  width: 100%;
  outline: none;
}

.speed-label {
  background: rgb(132, 0, 255);
  padding: 0.2em 1em;
  border-radius: 50px;
  color: #fff;
}

button {
  margin: 0.5em;
  padding: 0.5em 2em;
  border: none;
  border-radius: 3px;
  color: azure;
  cursor: pointer;
  font-weight: 600;
}

.btn-primary {
  background: rgb(92, 92, 255);
}
.btn-danger {
  background: rgb(255, 49, 49);
}

.btn-info {
  background: rgb(49, 255, 176);
}

.btn-disabled {
  background: lightgray;
  cursor: context-menu !important;
}

.baord {
  background: rgb(255, 255, 255);
  border: 1px solid;
}

.board-row {
  display: flex;
}

.board-col {
  width: 30px;
  height: 30px;
  background: rgb(255, 255, 255);
  display: flex;
  justify-content: center;
  align-items: center;
}

.snake {
  background: rgb(109, 255, 236);
}

.snake-head {
  background: rgb(0, 255, 191) !important;
}

.snake-eye {
  height: 3px;
  width: 3px;
  border-radius: 50px;
  background: rgb(136, 0, 0);
}

.snake-head-up {
  border-top-right-radius: 20px;
  border-top-left-radius: 20px;
}

.snake-head-down {
  border-bottom-right-radius: 20px;
  border-bottom-left-radius: 20px;
}

.snake-head-left {
  border-top-left-radius: 20px;
  border-bottom-left-radius: 20px;
}

.snake-head-right {
  border-top-right-radius: 20px;
  border-bottom-right-radius: 20px;
}

.snake-eye-rl {
  display: flex;
  flex-direction: column;
  justify-content: space-around;
}

.snake-eye-bt {
  display: flex;
  justify-content: space-around;
}

.fruit {
  background: rgb(255, 138, 109);
  border-radius: 50%;
  height: 20px;
  width: 20px;
}
</style>
