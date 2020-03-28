<template>
  <div class="dice-container">
      <div class="dice" :style="{ transform: transformDice }">
         <div class="front">
            <span></span>
            <span></span>
            <span></span>
            <span></span>
            <span></span>
            <span></span>
         </div>
         <div class="back">
            <span></span>
         </div>
         <div class="right">
            <span></span>
            <span></span>
            <span></span>
            <span></span>
            <span></span>
         </div>
         <div class="left">
            <span></span>
            <span></span>
         </div>
         <div class="top">
            <span></span>
            <span></span>
            <span></span>
         </div>
         <div class="bottom">
            <span></span>
            <span></span>
            <span></span>
            <span></span>
         </div>
      </div>
   </div>
</template>

<script>
export default {
  name: 'Dice',
  props: ['dice', 'rolled'],
  data() {
    return {
      currentIndex: 0,
      faces: [
        // Caras del dado 1 - 6
        ['rotateX(180deg) rotateY(0deg)', 'rotateX(-180deg) rotateY(0deg)'],
        ['rotateX(0deg) rotateY(90deg)', 'rotateX(360deg) rotateY(90deg)'],
        ['rotateX(270deg) rotateY(0deg)', 'rotateX(270deg) rotateY(360deg)'],
        ['rotateX(90deg) rotateY(0deg)', 'rotateX(90deg) rotateY(360deg)'],
        ['rotateX(180deg) rotateY(90deg)', 'rotateX(-180deg) rotateY(90deg)'],
        ['rotateX(0deg) rotateY(0deg)', 'rotateX(360deg) rotateY(360deg)'],
      ],
    };
  },
  computed: {
    transformDice() {
      return this.faces[this.dice - 1][this.currentIndex];
    },
  },
  watch: {
    rolled(rolled) {
      if (rolled) {
        this.currentIndex = Number(!this.currentIndex);
        setTimeout(() => {
          this.$emit('dice-rolled');
        }, 3000);
      }
    },
  },
};
</script>

<style lang="scss" scoped>
.dice {
  width: 60px;
  height: 60px;
  position: relative;
  transform-style: preserve-3d;
  transform: rotateX(0deg) rotateY(0deg);
  transition: transform 3s;

  > div {
    height: 60px;
    width: 60px;
    position: absolute;
    background: rgb(128, 184, 96);
  }

  span {
    width: 8px;
    height: 8px;
    background: #fff;
    border-radius: 50%;
    display: block;
    position: absolute;
  }

  .front {
    transform: rotateY(0deg) translateZ(30px);

    span {
      &:nth-child(1) {
        top: 10px;
        left: 12px;
      }
      &:nth-child(2) {
        top: 10px;
        right: 12px;
      }
      &:nth-child(3) {
        top: 26px;
        left: 12px;
      }
      &:nth-child(4) {
        top: 26px;
        right: 12px;
      }
      &:nth-child(5) {
        bottom: 10px;
        left: 12px;
      }
      &:nth-child(6) {
        bottom: 10px;
        right: 12px;
      }
    }
  }
  .back {
    transform: rotateX(180deg) translateZ(30px);

    span {
      top: 26px;
      left: 26px;
    }
  }
  .right {
    transform: rotateY(90deg) translateZ(30px);

    span {
      &:nth-child(1) {
        top: 12px;
        left: 12px;
      }
      &:nth-child(2) {
        top: 12px;
        right: 12px;
      }
      &:nth-child(3) {
        top: 26px;
        left: 26px;
      }
      &:nth-child(4) {
        bottom: 12px;
        left: 12px;
      }
      &:nth-child(5) {
        bottom: 12px;
        right: 12px;
      }
    }
  }
  .left {
    transform: rotateY(-90deg) translateZ(30px);

    span {
      &:nth-child(1) {
        top: 12px;
        right: 12px;
      }
      &:nth-child(2) {
        bottom: 12px;
        left: 12px;
      }
    }
  }
  .top {
    transform: rotateX(90deg) translateZ(30px);

    span {
      &:nth-child(1) {
        top: 12px;
        right: 12px;
      }
      &:nth-child(2) {
        bottom: 12px;
        left: 12px;
      }
      &:nth-child(3) {
        bottom: 26px;
        left: 26px;
      }
    }
  }
  .bottom {
    transform: rotateX(-90deg) translateZ(30px);

    span {
      &:nth-child(1) {
        top: 12px;
        right: 12px;
      }
      &:nth-child(2) {
        top: 12px;
        left: 12px;
      }
      &:nth-child(3) {
        bottom: 12px;
        left: 12px;
      }
      &:nth-child(4) {
        bottom: 12px;
        right: 12px;
      }
    }
  }
}

.button {
  position: fixed;
  bottom: 20px;
  background: #f76939;
  padding: 20px 40px;
  border-radius: 4px;
  color: #fff;
  cursor: pointer;

  &:hover {
    background: #f76939 - 20;
  }
}

@keyframes rotate {
  0% { transform: rotateX(90deg) rotateY(360deg) rotateZ(0deg) translateX(0); } // 4
  35% { transform: rotateX(-180deg) rotateY(-90deg) rotateZ(0deg) translateX(0); } // 2
  65% { transform: rotateX(0deg) rotateY(0deg) rotateZ(-360deg) translateX(0); } // 1
}
</style>
