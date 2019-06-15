<template>
  <div>
    <div class="header">
      <div>SCORE: {{ snakeLength }}</div>
    </div>
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
</template>

<script>
// TODO: Make sure logic for apple placement never puts apple on snake
// TODO: stop from moving back to previous block once you have a tail
//      maybe shake playing area to let player know
// TODO: Logic to check if snake head hits part of tail = GAME OVER
// TODO: Way to have the snake move automatically
//    Will continue in last direction moved unless user changes
//    running into wall will end game
//    slowly speed snake up

import { setInterval } from 'timers';

export default {
  name: 'home',
  data() {
    return {
      numRows: 12,
      numCols: 12,
      blocks: [],
      snakePreviousPosition: 0,
      snakePosition: 0,
      snakeLength: 0,
      snakeBlocksFound: 0,
      foundSnakes: [0],
      applePosition: 13,
      lastDirection: '',
      appleTimeout: 4000,
    };
  },
  created() {
    window.addEventListener('keydown', (event) => {
      switch (event.key) {
        case 'ArrowLeft':
          this.moveSnakeLeft();
          break;
        case 'ArrowRight':
          this.moveSnakeRight();
          break;
        case 'ArrowDown':
          this.moveSnakeDown();
          break;
        case 'ArrowUp':
          this.moveSnakeUp();
          break;
        default:
          break;
      }
    });
  },
  mounted() {
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
      }
    },
    moveSnakeDown() {
      if (this.snakePosition < ((this.numRows * this.numCols) - this.numCols)) {
        this.snakePreviousPosition = this.snakePosition;
        this.snakePosition = this.snakePosition + this.numCols;
        this.lastDirection = 'D';
        this.afterMove();
      }
    },
    moveSnakeLeft() {
      if ((this.snakePosition % this.numCols) - 1 >= 0) {
        this.snakePreviousPosition = this.snakePosition;
        this.snakePosition = this.snakePosition - 1;
        this.lastDirection = 'L';
        this.afterMove();
      }
    },
    moveSnakeUp() {
      if (this.snakePosition >= this.numCols) {
        this.snakePreviousPosition = this.snakePosition;
        this.snakePosition = this.snakePosition - this.numCols;
        this.lastDirection = 'U';
        this.afterMove();
      }
    },
    afterMove() {
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
      }
    },
    getSnakePosition() {
      const rowPosition = Math.floor(this.snakePosition / this.numRows);
      const colPostion = this.snakePosition % this.numCols;

      return {
        row: rowPosition,
        col: colPostion,
      };
    },
    classObject(index, block) {
      // start with the apple position class binding
      const styles = {
        apple: index === this.applePosition,
        snake: block === 'S',
      };
      return styles;
    },
  },
};
</script>

<style lang="scss" >
.grid {
  display: grid;
  grid-template-columns: repeat(12, 25px);
  grid-template-rows: repeat(12, 25px);
  grid-column-gap: 3px;
  grid-row-gap: 3px;
}

.block {
  background: #aaa;
}

.snake {
  background: greenyellow;
}

.apple {
  background: red;
}
</style>
