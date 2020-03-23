<template>
  <div class="home">
    <h2>The Quidditch Game</h2>
    <div id="board" ref="board">
      <div
        class="row"
        :class="{
          'row--alternate': !!(keyRow % 2),
          'row--goal': ~[0, board.inner.length - 1].indexOf(keyRow)
        }"
        v-for="(row, keyRow) in board.inner"
        :key="keyRow">
        <div
          class="space"
          :class="{
            'space--dead': !space,
            'space--alternate': !!(key % 2)
          }"
          v-for="(space, key) in row"
          :key="key">
        </div>
      </div>

      <div class="golden-snitch" ref="goldenSnitch">
        <div
          class="space"
          :class="{ 'space--alternate': !!(key % 2) }"
          :style="getSpaceStyleGoldenSnitch(key)"
          v-for="(space, key) in board.outer"
          :key="key">
        </div>
      </div>
    </div>
  </div>
</template>

<script>
/* eslint-disable indent */
/* eslint-disable array-bracket-spacing */
/* eslint-disable no-multi-spaces */
export default {
  name: 'Home',
  data() {
    return {
      board: {
        inner: [
          [ null, null, null, [], null, [], null, [], null, null, null ],
          [     null, null, [], [], [], [], [], [], [], null, null     ],
          [       null, [], [], [], [], [], [], [], [], [], null       ],
          [       null, [], [], [], [], [], [], [], [], [], null       ],
          [       null, [], [], [], [], [], [], [], [], [], null       ],
          [         [], [], [], [], [], [], [], [], [], [], []         ],
          [         [], [], [], [], [], [], [], [], [], [], []         ],
          [         [], [], [], [], [], [], [], [], [], [], []         ],
          [         [], [], [], [], [], [], [], [], [], [], []         ],
          [         [], [], [], [], [], [], [], [], [], [], []         ],
          [         [], [], [], [], [], [], [], [], [], [], []         ],
          [         [], [], [], [], [], [], [], [], [], [], []         ],
          [       null, [], [], [], [], [], [], [], [], [], null       ],
          [       null, [], [], [], [], [], [], [], [], [], null       ],
          [       null, [], [], [], [], [], [], [], [], [], null       ],
          [     null, null, [], [], [], [], [], [], [], null, null     ],
          [ null, null, null, [], null, [], null, [], null, null, null ],
        ],
        outer: new Array(76),
      },
      boardHeight: 0,
      boardWidth: 0,
    };
  },
  mounted() {
    this.boardHeight = parseInt(window.getComputedStyle(this.$refs.board).height.slice(0, -2), 10);
    this.boardWidth = parseInt(
      window.getComputedStyle(this.$refs.goldenSnitch).width.slice(0, -2), 10,
    );
  },
  methods: {
    getSpaceStyleGoldenSnitch(index) {
      const theta = ((360 / 76) / 180) * index * Math.PI;
      const posx = Math.round((this.boardWidth / 2) * (Math.cos(theta)));
      const posy = Math.round((this.boardHeight / 2) * (Math.sin(theta)));
      return {
        top: `${((this.boardHeight / 2) - parseInt(posy, 10))}px`,
        left: `${((this.boardWidth / 2) + parseInt(posx, 10))}px`,
      };
    },
  },
};
</script>

<style lang="scss">
  * {
    box-sizing: border-box;
  }

  h2 {
    font-family: cursive;
    text-align: center;
  }

  #board {
    position: relative;
    padding: 80px 0;
  }

  .golden-snitch {
    position: absolute;
    width: 50%;
    height: 100%;
    top: 0;
    transform: translateX(50%);

    .space {
      position: absolute;
      transform: translate(-50%, -50%);
      transition: all .5s;

      &:hover {
        transform: scale(1.5) translate(-50%, -50%);
        z-index: 999;
      }
    }
  }

  .space {
    width: 40px;
    height: 40px;
    background-color: green;
    display: inline-block;
    cursor: pointer;

    &--alternate {
      background-color: greenyellow;
    }
    &--dead {
      opacity: 0;
    }
    &--break {
      margin-right: auto;
    }
  }

  .row {
    text-align: center;

    &--alternate {
      .space {
        background-color: greenyellow;

        &--alternate {
          background-color: green;
        }
      }
    }

    &--goal {
      .space {
        background-color: yellow;
      }
    }
  }
</style>
