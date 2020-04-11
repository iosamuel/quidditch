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
        v-for="(row, keyRow) in board.inner"
        :key="keyRow"
        class="row"
        :class="{
          'row--alternate': !!(keyRow % 2),
          'row--goal': ~[0, board.inner.length - 1].indexOf(keyRow)
        }">
        <div
          v-for="(space, key) in row"
          :key="key"
          class="space"
          :class="{
            'space--dead': !space,
            'space--alternate': !!(key % 2)
          }"
          @click="space && movePiece(keyRow, key)">
          <template v-for="(chaser, pieceKey) in teams.team1.pieces.chasers">
            <Piece
              v-if="isInPosition(chaser.position, [keyRow, key])"
              :key="teams.team1.name + chaser.type + pieceKey"
              :piece="chaser"
              :space="[keyRow, key]"
              :team="teams.team1"
              :pieceMoving="pieceMoving"
              @click="!gameBegin.init && pieceClicked(chaser, 'team1')"/>
          </template>
          <template v-for="(chaser, pieceKey) in teams.team2.pieces.chasers">
            <Piece
              v-if="isInPosition(chaser.position, [keyRow, key])"
              :key="teams.team2.name + chaser.type + pieceKey"
              :piece="chaser"
              :space="[keyRow, key]"
              :team="teams.team2"
              :pieceMoving="pieceMoving"
              @click="!gameBegin.init && pieceClicked(chaser, 'team2')"/>
          </template>
          <template v-for="(beater, pieceKey) in teams.team1.pieces.beaters">
            <Piece
              v-if="isInPosition(beater.position, [keyRow, key])"
              :key="teams.team1.name + beater.type + pieceKey"
              :piece="beater"
              :space="[keyRow, key]"
              :team="teams.team1"
              :pieceMoving="pieceMoving"
              @click="!gameBegin.init && pieceClicked(beater, 'team1')"/>
          </template>
          <template v-for="(beater, pieceKey) in teams.team2.pieces.beaters">
            <Piece
              v-if="isInPosition(beater.position, [keyRow, key])"
              :key="teams.team2.name + beater.type + pieceKey"
              :piece="beater"
              :space="[keyRow, key]"
              :team="teams.team2"
              :pieceMoving="pieceMoving"
              @click="!gameBegin.init && pieceClicked(beater, 'team2')"/>
          </template>
          <Piece
            v-if="isInPosition(teams.team1.pieces.keeper.position, [keyRow, key])"
            :piece="teams.team1.pieces.keeper"
            :space="[keyRow, key]"
            :team="teams.team1"
            :pieceMoving="pieceMoving"
            @click="!gameBegin.init && pieceClicked(teams.team1.pieces.keeper, 'team1')"/>
          <Piece
            v-if="isInPosition(teams.team2.pieces.keeper.position, [keyRow, key])"
            :piece="teams.team2.pieces.keeper"
            :space="[keyRow, key]"
            :team="teams.team2"
            :pieceMoving="pieceMoving"
            @click="!gameBegin.init && pieceClicked(teams.team2.pieces.keeper, 'team2')"/>
          <Piece
            v-if="isInPosition(teams.quaffle.position, [keyRow, key])"
            :gameInit="!gameBegin.init"
            :piece="teams.quaffle"
            :space="[keyRow, key]"/>
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
            ),
            'space--sitting': ~[
                teams.team1.pieces.goldenSnitch.position,
                teams.team2.pieces.goldenSnitch.position
              ].indexOf(key),
          }"
          :style="getSpaceStyleGoldenSnitch(key)"
          v-for="(space, key) in goldenSnitchSpaces"
          :key="key">
          <svg v-if="
              key === teams.team1.pieces.goldenSnitch.goalIndex ||
              key === teams.team2.pieces.goldenSnitch.goalIndex"
              version="1.1" id="Capa_1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" x="0px" y="0px"
              viewBox="0 0 100 100" style="enable-background:new 0 0 100 100;" xml:space="preserve">
            <circle cx="50" cy="50" r="50"/>
              <g>
                <path class="st0" d="M64.3,11.4c2.7,24.7-4,32.4-4,32.4L47.7,61.1c4,1.5,7.1,4.7,8.6,8.7l18.2-13C83.6,38.8,64.3,11.4,64.3,11.4z"
                  />
                <path class="st1" d="M47.7,61.1c-1.7-0.6-3.4-1-5.3-1c-1.5,0-3,0.2-4.4,0.7l-0.1,2.4L32.4,69l-1-4c-2.4,2.7-3.9,6.2-3.9,10.1
                  c0,8.2,6.7,14.9,14.9,14.9s14.9-6.7,14.9-14.9c0-1.8-0.3-3.6-0.9-5.2C54.9,65.9,51.7,62.7,47.7,61.1z"/>
                <path class="st0" d="M37.9,63.2l0.1-2.4l0.1-2.6c0,0,5.5-8.6,7.3-10.7c1.8-2.1-2-14.3-4.4-20.3C38.6,21.2,23,10,23,10
                  c12,12.8,6,34.1,6,34.1c-1.8,7.5-0.2,9.9-0.2,9.9l2.7,11l1,4L37.9,63.2z"/>
              </g>
          </svg>
          <div
            class="piece piece--golden-snitch"
            v-if="key === teams.team1.pieces.goldenSnitch.position ||
                  key === teams.team2.pieces.goldenSnitch.position">
            <svg version="1.1" id="Capa_1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" x="0px" y="0px"
                viewBox="0 0 100 100" style="enable-background:new 0 0 100 100;" xml:space="preserve">
              <circle
                cx="50" cy="50" r="50"
                :style="{ fill: key === teams.team1.pieces.goldenSnitch.position ? teams.team1.color : teams.team2.color }"/>
              <g style="fill: white">
                <g>
                  <path class="st0" d="M50,72c-17.7,0-34.4-7.8-45.8-21.4L3.6,50l0.6-0.7C15.6,35.7,32.3,28,50,28c17.7,0,34.4,7.8,45.8,21.4
                    l0.6,0.7l-0.6,0.7C84.4,64.3,67.7,72,50,72z M6.4,50c11,12.7,26.8,20,43.6,20c16.8,0,32.7-7.3,43.6-20C82.7,37.3,66.8,30,50,30
                    C33.2,30,17.3,37.3,6.4,50z"/>
                </g>
                <g>
                  <path class="st0" d="M50.3,33.6c-9.2,0-16.6,7.5-16.6,16.6s7.5,16.6,16.6,16.6s16.6-7.5,16.6-16.6S59.5,33.6,50.3,33.6z
                    M50.3,57.1c-3.8,0-6.9-3.1-6.9-6.9c0-3.8,3.1-6.9,6.9-6.9s6.9,3.1,6.9,6.9C57.2,54,54.1,57.1,50.3,57.1z"/>
                  <path class="st0" d="M50.3,67.2c-9.4,0-17-7.6-17-17s7.6-17,17-17s17,7.6,17,17S59.6,67.2,50.3,67.2z M50.3,33.9
                    c-9,0-16.3,7.3-16.3,16.3s7.3,16.3,16.3,16.3s16.3-7.3,16.3-16.3S59.3,33.9,50.3,33.9z M50.3,57.5c-4,0-7.3-3.3-7.3-7.3
                    s3.3-7.3,7.3-7.3s7.3,3.3,7.3,7.3S54.3,57.5,50.3,57.5z M50.3,43.6c-3.6,0-6.6,3-6.6,6.6c0,3.6,3,6.6,6.6,6.6c3.6,0,6.6-3,6.6-6.6
                    C56.9,46.6,53.9,43.6,50.3,43.6z"/>
                </g>
              </g>
            </svg>
          </div>
        </div>
      </div>
    </div>
    <div class="panel">
      <div class="panel__message" v-if="!gameBegin.init">
        Turno de:
        <span
          :style="{ color: teams[turn].color }">
          {{ teams[turn].name }}
        </span>
      </div>
      <div class="panel__message" v-else>
        Tiren los dados para saber quien empieza.
      </div>
      <button
        class="btn"
        :style="{ backgroundColor: teams[turn].color }"
        @click="rollDiceTeam"
        :disabled="!gameBegin.init || diceRolled">
        {{ teams[turn].name }}: Roll the dice.
      </button>
      <button
        class="btn"
        @click="rollGoldenSnitch"
        :disabled="diceRolled || !canGoldenSnitch">
        Roll to Golden Snitch!
      </button>
      <Dice :dice="dice" :rolled="diceRolled" ref="diceElement" />
    </div>
  </div>
</template>

<script>
/* eslint-disable indent */
/* eslint-disable array-bracket-spacing */
/* eslint-disable no-multi-spaces */
import Alerts from '@/components/Alerts.vue';
import Dice from '@/components/Dice.vue';
import Piece from '@/components/Piece.vue';

let resize;

const goldenSnitchSpaces = 76;
const halfGoldenSnitchSpaces = (goldenSnitchSpaces / 2);

// TODO: Agregar cartas de juego
// TODO:! Agregar las reglas de movimiento

// types:chaser
const chaser = {
  type: 'chaser',
};

// types:beater
const beater = {
  type: 'beater',
};

// types:keeper
const keeper = {
  type: 'keeper',
  rules: {
    y: 1,
    x: 1,
  },
};

// types:quaffle
const quaffle = {
  type: 'quaffle',
  position: { x: 5, y: 8 },
  startPosition: { x: 5, y: 8 },
};

const team1Structure = {
  pieces: {
    goldenSnitch: {
      position: 0,
      startIndex: 0,
      goalIndex: halfGoldenSnitchSpaces / 2,
    },
    chasers: [
      {
        ...chaser,
        position: { x: 5, y: 6 },
        startPosition: { x: 5, y: 6 },
      },
      {
        ...chaser,
        position: { x: 4, y: 5 },
        startPosition: { x: 4, y: 5 },
      },
      {
        ...chaser,
        position: { x: 6, y: 5 },
        startPosition: { x: 6, y: 5 },
      },
    ],
    beaters: [
      {
        ...beater,
        position: { x: 4, y: 4 },
        startPosition: { x: 4, y: 4 },
      },
      {
        ...beater,
        position: { x: 6, y: 4 },
        startPosition: { x: 6, y: 4 },
      },
    ],
    keeper: {
      ...keeper,
      position: { x: 5, y: 0 },
      startPosition: { x: 5, y: 0 },
      rules: {
        ...keeper.rules,
        minY: 0,
        maxY: 1,
      },
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
    chasers: [
      {
        ...chaser,
        position: { x: 5, y: 10 },
        startPosition: { x: 5, y: 10 },
      },
      {
        ...chaser,
        position: { x: 4, y: 11 },
        startPosition: { x: 4, y: 11 },
      },
      {
        ...chaser,
        position: { x: 6, y: 11 },
        startPosition: { x: 6, y: 11 },
      },
    ],
    beaters: [
      {
        ...beater,
        position: { x: 4, y: 12 },
        startPosition: { x: 4, y: 12 },
      },
      {
        ...beater,
        position: { x: 6, y: 12 },
        startPosition: { x: 6, y: 12 },
      },
    ],
    keeper: {
      ...keeper,
      position: { x: 5, y: 16 },
      startPosition: { x: 5, y: 16 },
      rules: {
        ...keeper.rules,
        minY: 15,
        maxY: 16,
      },
    },
  },
};

export default {
  name: 'Game',
  components: {
    Alerts,
    Dice,
    Piece,
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
          color: '#234a45',
          ...JSON.parse(JSON.stringify(team1Structure)),
        },
        team2: {
          name: 'Gryffindor',
          color: '#512830',
          ...JSON.parse(JSON.stringify(team2Structure)),
        },
        quaffle,
      },
      goldenSnitchSpaces,
      turn: 'team1',
      winner: null,
      alert: {
        show: false,
        message: '',
        type: '',
      },
      dice: 1,
      diceRolled: false,
      pieceMoving: null,
      gameBegin: {
        init: true,
        team1: null,
        team2: null,
      },
    };
  },
  computed: {
    canGoldenSnitch() {
      // TODO: Agregar logica para que el equipo tire a golden snitch
      return false;
    },
  },
  mounted() {
    resize = new ResizeObserver(() => {
      this.board.boardHeight = parseInt(
        window.getComputedStyle(this.$refs.board).height.slice(0, -2), 10,
      );
      this.board.boardWidth = parseInt(
        window.getComputedStyle(this.$refs.goldenSnitch).width.slice(0, -2), 10,
      );
    });
    resize.observe(this.$refs.board);
  },
  beforeDestroy() {
    resize.unobserve(this.$refs.board);
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
      this.dice = Math.floor(Math.random() * (6 - 1)) + 1; // 6 numeros del dado
      this.diceRolled = true;
      return new Promise((resolve) => {
        this.$refs.diceElement.$once('dice-rolled', () => {
          this.diceRolled = false;
          resolve();
        });
      });
    },
    rollDiceTeam() {
      this.rollDice().then(() => {
        if (this.gameBegin.init) {
          this.gameBegin[this.turn] = this.dice;
          if (this.gameBegin.team1 && this.gameBegin.team2) {
            if (this.gameBegin.team1 === this.gameBegin.team2) {
              // No pueden ser iguales, toca repetir el proceso
              this.gameBegin.team1 = null;
              this.gameBegin.team2 = null;
              this.changeTurn();
            } else {
              this.turn = (
                this.gameBegin.team1 > this.gameBegin.team2
                ? 'team1'
                : 'team2'
              );
              this.gameBegin.init = false;
              this.quaffleMove(true);
            }
          } else {
            this.changeTurn();
          }
        }
      });
    },
    rollGoldenSnitch() {
      this.rollDice().then(() => {
        const currentPosition = this.teams[this.turn].pieces.goldenSnitch.position;
        const goal = this.teams[this.turn].pieces.goldenSnitch.goalIndex;
        const futurePosition = currentPosition + this.dice;

        if (futurePosition <= goal) {
          this.teams[this.turn].pieces.goldenSnitch.position += this.dice;
        } else if (futurePosition === goal) {
          this.teams[this.turn].pieces.goldenSnitch.position = goal;
        } else {
          this.showAlert(`Muy alto! Necesitas un ${goal - currentPosition}`, 'error');
          this.changeTurn();
        }
      });
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

    // Pieces
    pieceClicked(piece, team) {
      if (this.pieceMoving) {
        if (this.pieceMoving === piece) {
          this.pieceMoving = null;
        }
      } else if (this.turn === team) {
        this.pieceMoving = piece;
      }
    },
    isInPosition(piecePosition, [ y, x ]) {
      return piecePosition.x === x && piecePosition.y === y;
    },
    movePiece(y, x) {
      if (this.pieceMoving && this.pieceMoving.type === 'keeper') {
        const { x: currentPositionX, y: currentPositionY } = this.pieceMoving.position;
        const {
          x: rulesX,
          y: rulesY,
          minY,
          maxY,
        } = this.pieceMoving.rules;
        const resPositionX = Math.abs(x - currentPositionX);
        const resPositionY = Math.abs(y - currentPositionY);
        if (
          (minY === y || maxY === y)
          && resPositionX === rulesX
          && resPositionY === rulesY
        ) {
          this.pieceMoving.position.x = x;
          this.pieceMoving.position.y = y;

          this.changeTurn();
          this.pieceMoving = null;
        } else {
          // mostrar alerta error? no se puede mover a (y, x)
        }
      }
    },
    quaffleMove(begin) {
      if (begin) {
        this.teams.quaffle.position = this.teams[this.turn].pieces.chasers[0].position;
      } else {
        // TODO: Algoritmo de busqueda, chaser mas cercano al quaffle
      }
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
