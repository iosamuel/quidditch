<template>
  <div class="game">
    <h2>The Quidditch Game</h2>
    <Alerts
      :show="alert.show"
      :alert="alert"
      @show="alert.show = $event"
      @reset="resetGame()"/>
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
          :class="{
            'space--alternate': !!(key % 2),
            'space--start': (
              key === teams.team1.pieces.goldenSnitch.startIndex ||
              key === teams.team2.pieces.goldenSnitch.startIndex
            ),
            'space--goal': (
              key === teams.team1.pieces.goldenSnitch.goalIndex ||
              key === teams.team2.pieces.goldenSnitch.goalIndex
            )
          }"
          :style="getSpaceStyleGoldenSnitch(key)"
          v-for="(space, key) in goldenSnitchSpaces"
          :key="key">
          <div
            class="piece piece--golden-snitch"
            v-if="key === teams.team1.pieces.goldenSnitch.position">
          </div>
          <div
            class="piece piece--golden-snitch"
            v-if="key === teams.team2.pieces.goldenSnitch.position">
          </div>
        </div>
      </div>
    </div>
    <hr>
    <button @click="rollGoldenSnitch">Roll to Golden Snitch!</button>
  </div>
</template>

<script>
/* eslint-disable indent */
/* eslint-disable array-bracket-spacing */
/* eslint-disable no-multi-spaces */
import Alerts from '@/components/Alerts.vue';

const goldenSnitchSpaces = 76;
const halfGoldenSnitchSpaces = (goldenSnitchSpaces / 2);

const team1Structure = {
  pieces: {
    goldenSnitch: {
      position: 0,
      startIndex: 0,
      goalIndex: halfGoldenSnitchSpaces / 2,
    },
  },
};
const team2Structure = {
  pieces: {
    goldenSnitch: {
      position: halfGoldenSnitchSpaces,
      startIndex: halfGoldenSnitchSpaces,
      goalIndex: (halfGoldenSnitchSpaces / 2) + halfGoldenSnitchSpaces,
    },
  },
};

export default {
  name: 'Game',
  components: {
    Alerts,
  },
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
        boardHeight: 0,
        boardWidth: 0,
      },
      teams: {
        team1: {
          name: 'Slytherin',
          ...JSON.parse(JSON.stringify(team1Structure)),
        },
        team2: {
          name: 'Hogwarts',
          ...JSON.parse(JSON.stringify(team2Structure)),
        },
      },
      goldenSnitchSpaces,
      turn: 'team1',
      winner: null,
      alert: {
        show: false,
        message: '',
        type: '',
      },
    };
  },
  mounted() {
    this.board.boardHeight = parseInt(
      window.getComputedStyle(this.$refs.board).height.slice(0, -2), 10,
    );
    this.board.boardWidth = parseInt(
      window.getComputedStyle(this.$refs.goldenSnitch).width.slice(0, -2), 10,
    );
  },
  methods: {
    getSpaceStyleGoldenSnitch(index) {
      const theta = ((360 / 76) / 180) * index * Math.PI;
      const posx = Math.round((this.board.boardWidth / 2) * (Math.cos(theta)));
      const posy = Math.round((this.board.boardHeight / 2) * (Math.sin(theta)));
      return {
        top: `${((this.board.boardHeight / 2) - parseInt(posy, 10))}px`,
        left: `${((this.board.boardWidth / 2) + parseInt(posx, 10))}px`,
      };
    },
    rollDice() {
      return Math.floor(Math.random() * (6 - 1)) + 1; // 6 numeros del dado
    },
    rollGoldenSnitch() {
      const dice = this.rollDice();
      const currentPosition = this.teams[this.turn].pieces.goldenSnitch.position;
      const goal = this.teams[this.turn].pieces.goldenSnitch.goalIndex;
      const futurePosition = currentPosition + dice;

      if (futurePosition <= goal) {
        this.teams[this.turn].pieces.goldenSnitch.position += dice;
      } else if (futurePosition === goal) {
        this.teams[this.turn].pieces.goldenSnitch.position = goal;
      } else {
        this.showAlert(`Muy alto! Necesitas un ${goal - currentPosition}`, 'error');
        this.changeTurn();
      }
    },
    showAlert(message, type) {
      this.alert = {
        show: true,
        message,
        type,
      };
    },
    check() {
      this.winCondition();
      this.changeTurn();
    },
    winCondition() {
      if (
        this.teams[this.turn].pieces.goldenSnitch.position
        === this.teams[this.turn].pieces.goldenSnitch.goalIndex
      ) {
        this.winner = this.teams[this.turn].name;
        this.showAlert(`Equipo ${this.winner} ha GANADO!`, 'win');
      }
    },
    changeTurn() {
      this.turn = this.turn === 'team1' ? 'team2' : 'team1';
    },
    resetGame() {
      this.teams.team1 = {
        ...this.teams.team1,
        ...team1Structure,
      };
      this.teams.team2 = {
        ...this.teams.team2,
        ...team2Structure,
      };
      this.winner = null;
    },
  },
  watch: {
    'teams.team1.pieces.goldenSnitch.position': function handler() {
      this.check();
    },
    'teams.team2.pieces.goldenSnitch.position': function handler() {
      this.check();
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

  .piece {
    position: absolute;
    display: inline-block;
    width: 75%;
    height: 75%;
    border-radius: 50%;
    border: 1px solid white;

    &--golden-snitch {
      background: black;
    }
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

      &--start,
      &--goal {
        &::before {
          content: ' ';
          display: inline-block;
          width: 75%;
          height: 75%;
          border: 1px solid white;
          border-radius: 50%;
        }
      }

      &--goal::before {
        background: white;
      }
    }
  }

  .space {
    width: 40px;
    height: 40px;
    background-color: green;
    display: inline-flex;
    justify-content: center;
    align-items: center;
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
