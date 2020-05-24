<template>
  <div>
    <div><input type="button" value="reset" @click="resetBoard" /></div>
    <div class="minefield">
      <div v-for="y in dim" :key="y">
        <div v-for="x in dim" :key="x">
          <Tile :density="density"
            :rowidx="y-1"
            :colidx="x-1">
          </Tile>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import eventBus from './EventBus'
import Tile from './Tile'

export default {
  name: "MindField",
  components: {
    Tile 
  },
  props: {
    dim: {
      type: Number,
      default: 10
    },
    density: {
      type: Number,
      default: 0.15
    }
  },
  data() {
    return {
      total: 0
    };
  },
  methods: {
    resetBoard() {
      this.total=0
      eventBus.$emit('broadcast','reset');
      setTimeout(function() {
            eventBus.$emit('broadcast','iamabomb');
      },200)
    },
    checkWon() {
      var won = true;
      for (var ridx=0; ridx<this.dim; ridx++) {
        for (var cidx=0; cidx<this.dim; cidx++) {
          var cell = this.board[ridx][cidx];
          if (cell.isBomb && !(cell.isClosed||cell.isFlagged)) {
            won = false
            break;
          } else if (!cell.isBomb && cell.isClosed) {
            won = false
            break;
          }
        }
        if (won==false) {
          break;
        }
      }
      if (won) {
        alert("You WON!!");
      }
    },
    addToTotal() {
      this.total++;
      if (this.total==this.dim*this.dim) {
        alert('You Won');
      }
    }
  },
  created() {
    this.resetBoard();
    eventBus.$on('opened', this.addToTotal)
    eventBus.$on('iamaflag', this.addToTotal)
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.minefield {
  display: flex;
  justify-content: center;
}

</style>
