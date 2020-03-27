<template>
  <div class="message"
    v-if="show"
    @click="alert.type !== 'win' ? $emit('show', !show) : null">
    <span>{{ alert.message }}</span>
    <button
      v-if="alert.type === 'win'"
      class="reset"
      @click="$emit('reset') && $emit('show', !show)">
      Reset Game
    </button>
  </div>
</template>

<script>
export default {
  name: 'Alerts',
  props: ['show', 'alert'],
};
</script>

<style lang="scss" scoped>
.message {
  position: absolute;
  top: 0;
  right: 0;
  left: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  z-index: 9999;
  font-size: 50px;
  font-weight: bold;
  color: white;

  .reset {
    padding: 16px;
    background-color: white;
    border-radius: 8px;
    border: none;
    cursor: pointer;
    box-shadow: 2px 2px 2px black;
    font-size: 18px;

    &:hover {
      background-color: whitesmoke;
    }
  }
}
</style>
