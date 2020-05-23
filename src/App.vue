<template>
  <div id="app">
    <div><h1>Minesweeper</h1></div>
    <div>Dimension: <input type="button" @click="decrease" value="-" /> {{dimension}} <input type="button" @click="increase" value="+" /></div>
    <div>Difficulty: <input type="range" v-model.number="dense" /></div>{{difficulty}}
    <MineField :dim="dimension" :density="dense/250"></MineField>
  </div>
</template>

<script>
import MineField from './components/MineField.vue'

export default {
  name: 'App',
  components: {
    MineField
  },
  data() {
    return {
      dimension: 15,
      dense: 10
    }
  },
  computed: {
    difficulty() {
      if (this.dense<25) {
        return 'Easy'
      } else if (this.dense<40) {
        return 'Regular'
      } else if (this.dense<60) {
        return 'Intermediate'
      } else if (this.dense<80) {
        return 'Difficult'
      } else {
        return 'Insane'
      }
    }
  },
  methods: {
    increase() {
      this.dimension++;
    },
    decrease() {
      if (this.dimension>3) {
        this.dimension--;
      }
    },
    increaseD() {
      if (this.dense < 0.50) {
        alert(this.dense)
        this.dense = (this.dense+0.01);
      }
    },
    decreaseD() {
      if (this.dense>0.01) {
        this.dense = (this.dense-0.01);
      }
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
