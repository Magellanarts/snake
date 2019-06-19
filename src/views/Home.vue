<template>
  <div>
    <div class="header">
      <div>SCORE: {{ snakeLength }}</div>
      <div>
        <button @click="increaseGridSize">Increase</button>
        <button @click="decreaseGridSize">Decrease</button>
        <button @click="pauseGame">Pause</button>
      </div>
    </div>

    <div>
      <h4>LEGEND</h4>
      <div class="legend">
        <div class="legend__snake"></div>
        <div>Snake</div>
        <div class="legend__apple"></div>
        <div>Apple</div>
      </div>
    </div>
    <div class="game-container">
      <div v-if="gameOver" class="game-over">GAME OVER</div>
      <div class="grid" ref="grid">
        <div
          v-for="(block, index) in blocks"
          class="block"
          :key="index"
          :class="classObject(index, block)"
        >
        {{ index }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// TODO: Make sure logic for apple placement never puts apple on snake
// TODO: Arrow buttons for small screen
// TODO: color of snake should be a gradient from head to tail

import { setInterval, setTimeout, clearTimeout } from 'timers';

export default {
  name: 'home',
  data() {
    return {
      numRows: 11,
      numCols: 11,
      blocks: [],
      snakePreviousPosition: 0,
      snakePosition: 0,
      snakeLength: 0,
      snakeBlocksFound: 0,
      foundSnakes: [0],
      applePosition: 13,
      lastDirection: '',
      appleTimeout: 3500,
      hasMoved: false,
      moveTimeout: 300,
      gameOver: false,
      movementStart: false,
      gameTimeout: '',
    };
  },
  created() {
    window.addEventListener('keydown', (event) => {
      switch (event.key) {
        case 'ArrowLeft':
          if (this.movementStart === false) {
            this.movementStart = true;
            this.moveSnakeLeft();
          } else {
            this.lastDirection = 'L';
          }
          break;
        case 'ArrowRight':
          if (this.movementStart === false) {
            this.movementStart = true;
            this.moveSnakeRight();
          } else {
            this.lastDirection = 'R';
          }
          break;
        case 'ArrowDown':
          if (this.movementStart === false) {
            this.movementStart = true;
            this.moveSnakeDown();
          } else {
            this.lastDirection = 'D';
          }
          break;
        case 'ArrowUp':
          if (this.movementStart === false) {
            this.movementStart = true;
            this.moveSnakeUp();
          } else {
            this.lastDirection = 'U';
          }
          break;
        default:
          break;
      }
    });
  },
  mounted() {
    // set up grid
    this.$refs.grid.style.gridTemplateColumns = `repeat(${this.numCols}, 25px)`;
    this.$refs.grid.style.gridTemplateRows = `repeat(${this.numRows}, 25px)`;

    this.blocks = new Array(this.numRows * this.numCols);
    this.blocks[this.snakePosition] = 'S';

    setInterval(() => {
      if (!this.applePosition) {
        let tempApplePosition = this.setApplePosition();
        if (this.foundSnakes.includes(tempApplePosition)) {
          tempApplePosition = this.setApplePosition();
        } else {
          this.applePosition = tempApplePosition;
        }
      }
    }, this.appleTimeout);
  },
  methods: {
    setApplePosition() {
      return Math.floor(Math.random(0, 1) * this.numRows * this.numCols);
    },
    moveSnakeRight() {
      if ((this.snakePosition + 1) % this.numCols > 0) {
        this.snakePreviousPosition = this.snakePosition;
        this.snakePosition = this.snakePosition + 1;

        this.lastDirection = 'R';
        this.afterMove();
      } else {
        this.endGame();
      }
    },
    moveSnakeDown() {
      if (this.snakePosition < ((this.numRows * this.numCols) - this.numCols)) {
        this.snakePreviousPosition = this.snakePosition;
        this.snakePosition = this.snakePosition + this.numCols;
        this.lastDirection = 'D';
        this.afterMove();
      } else {
        this.endGame();
      }
    },
    moveSnakeLeft() {
      if ((this.snakePosition % this.numCols) - 1 >= 0) {
        this.snakePreviousPosition = this.snakePosition;
        this.snakePosition = this.snakePosition - 1;
        this.lastDirection = 'L';
        this.afterMove();
      } else {
        this.endGame();
      }
    },
    moveSnakeUp() {
      if (this.snakePosition >= this.numCols) {
        this.snakePreviousPosition = this.snakePosition;
        this.snakePosition = this.snakePosition - this.numCols;
        this.lastDirection = 'U';
        this.afterMove();
      } else {
        this.endGame();
      }
    },
    afterMove() {
      // check to see if this new position already contains the snake
      // if so, game over
      if (this.blocks[this.snakePosition] === 'S') {
        this.endGame();
      }

      // set the index of the head of snake in the blocks array to an S
      this.$set(this.blocks, this.snakePosition, 'S');

      // add the position of the head to the foundSnakes array
      this.foundSnakes.push(this.snakePosition);

      // check for snakes
      // updates the tail of snake
      // if the length of snake + 1 (snake length starts as 0) is <
      // length of total blocks containging an S, remove the first item in array
      if (this.snakeLength + 1 < this.foundSnakes.length) {
        this.$set(this.blocks, this.foundSnakes[0], '');
        this.foundSnakes.splice(0, 1);
      }


      // if the head of snake meets an apple
      if (this.snakePosition === this.applePosition) {
        // increase snake length
        this.snakeLength = this.snakeLength + 1;
        // reset apple position
        // / interval will set a new one
        this.applePosition = '';

        // decrease automovement timeout
        this.moveTimeout = this.moveTimeout * 0.98;
      }

      // start automovement
      if (!this.hasMoved) {
        this.hasMoved = true;
        const autoMovement = () => {
          switch (this.lastDirection) {
            case 'R':
              this.moveSnakeRight();
              break;
            case 'D':
              this.moveSnakeDown();
              break;
            case 'L':
              this.moveSnakeLeft();
              break;
            case 'U':
              this.moveSnakeUp();
              break;
            default:
              break;
          }

          this.gameTimeout = setTimeout(autoMovement, this.moveTimeout);
        };

        setTimeout(autoMovement, this.moveTimeout);
      }
    },
    endGame() {
      this.gameOver = true;
    },
    classObject(index, block) {
      // start with the apple position class binding
      const styles = {
        apple: index === this.applePosition,
        snake: block === 'S',
      };
      return styles;
    },
    increaseGridSize() {
      this.numRows = this.numRows + 1;
      this.numCols = this.numCols + 1;

      this.$refs.grid.style.gridTemplateColumns = `repeat(${this.numCols}, 25px)`;
      this.$refs.grid.style.gridTemplateRows = `repeat(${this.numRows}, 25px)`;

      const tempArray = new Array((this.numRows * this.numCols) - (((this.numRows - 1) * (this.numCols - 1))));

      this.blocks.push(...tempArray);
    },
    decreaseGridSize() {
      this.numRows = this.numRows - 1;
      this.numCols = this.numCols - 1;

      this.$refs.grid.style.gridTemplateColumns = `repeat(${this.numCols}, 25px)`;
      this.$refs.grid.style.gridTemplateRows = `repeat(${this.numRows}, 25px)`;

      const tempArray = new Array((this.numRows * this.numCols) - (((this.numRows - 1) * (this.numCols - 1))));

      this.blocks.push(...tempArray);
    },
    pauseGame() {
      console.log('pasuin');
      clearTimeout(this.gameTimeout);
    },
  },
};
</script>

<style lang="scss">

.grid {
  display: grid;
 // grid-template-columns: repeat(12, 25px);
 // grid-template-rows: repeat(12, 25px);
  grid-column-gap: 3px;
  grid-row-gap: 3px;
}

.legend {
  grid-template-columns: 25px calc(100% - 25px);
  display: grid;
  grid-column-gap: 3px;
  grid-row-gap: 3px;
  max-width: 300px;
  margin-bottom: 15px;

  &__apple {
    background: red;
  }

  &__snake {
    background: greenyellow;
  }
}

.block {
  background: #aaa;
  transition: .1s linear;
}

.snake {
  background: greenyellow;
}

.apple {
  background: red;
}

.game-container {
  position: relative;
}

.game-over {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  margin: auto;
  font-size: 28px;
  background: #fff;
}
</style>
