<template>
  <div>
    <div><input type="button" value="reset" @click="resetBoard" /></div>
    <div class="minefield">
      <div v-for="(y, rowidx) in board" :key="rowidx">
        <div v-for="(x, colidx) in y" :key="colidx">
          <div class="cell">
            <i class="fas fa-bomb" v-if="x.isBomb"></i>
            <i class="hint" v-else-if="x.solution > 0" v-on:dblclick="clearCell(rowidx, colidx)">{{x.solution}}</i>
            <i class="fas fa-stop" style="color:white;" v-else></i>
          </div>
          <span class="fa-stack fa-2x cover" v-bind:id="rowidx + '-' + colidx"
            v-on:click="openCell(rowidx, colidx, $event)"
            v-on:click.right="flagCell(rowidx, colidx, $event)"
            @contextmenu.prevent>
          <i class="fas fa-stop fa-stack-2x"></i>
          <i class="fas fa-flag-checkered fa-stack-1x fa-inverse" v-show="x.isFlagged"></i>
          </span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "MindField",
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
      board: [],
      touchHandler: {
        row: 0,
        col: 0,
        handler: null
      }
    };
  },
  methods: {
    allTileCovers(show) {
      var allCovers = null
      if (show) {
        allCovers = document.querySelectorAll(".reveal");
        if (allCovers) {
          for (var cidx=0; cidx<allCovers.length; cidx++) {
            if (allCovers[cidx].classList) {
              allCovers[cidx].classList.remove("reveal")
            }
          }
        }
      } else {
        allCovers = document.querySelectorAll(".cover");
        if (allCovers) {
          for (cidx=0; cidx<allCovers.length; cidx++) {
            allCovers[cidx].classList.add("reveal")
          }
        }
      }
    },
    resetBoard() {
      this.allTileCovers(true);
      var allFlags = document.querySelectorAll(".fa-flag-checkered");
      if (allFlags) {
        for (var flag=0; flag<allFlags.length; flag++) {
          allFlags[flag].style.display='none'
        }
      }
      this.board = new Array(this.dim)
      for (var i=0; i<this.board.length; i++) {
        var row = new Array(this.dim);
        for (var j=0; j<row.length; j++) {
          row[j] = {
            isBomb: (Math.random()<this.density),
            solution: 0,
            isClosed: true,
            isFlagged: false
          };
        }
        this.board[i] = row;
      }
      for (var ridx=0; ridx<this.dim; ridx++) {
        for (var cidx=0; cidx<this.dim; cidx++) {
          var answer=0;
          for (var u=ridx-1; u<=ridx+1; u++) {
            // up, middle, bottom; relative to current cell
            if (u<0 || u==this.dim) continue;
            for (var lr=cidx-1; lr<=cidx+1; lr++) {            
              // left, center, right; relative to current cell
              if (lr<0 || lr==this.dim) continue;
                if (this.board[u][lr].isBomb) {
                  answer++
                }
            }
          }
          this.board[ridx][cidx].solution=answer
        }
      }
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
    openSurroundingCell(row, col) {
      for (var u=row-1; u<=row+1; u++) {
          // up, middle, bottom; relative to current cell
          if (u<0 || u==this.dim) continue;
          for (var lr=col-1; lr<=col+1; lr++) {            
            // left, center, right; relative to current cell
            if (lr<0 || lr==this.dim) continue;
            if (this.board[u][lr].isClosed && !this.board[u][lr].isFlagged) {
              this.openCell(u, lr, null);
            }
          }
      }
    },
    openCell(row, col, event) {
      var target = null;
      if (event) {
        target = event.currentTarget
      } else {
        target=document.getElementById(row+'-'+col);
      }
      if (!this.board[row][col].isFlagged) {
        this.board[row][col].isClosed = false;
        target.classList.add('reveal')
        if (this.board[row][col].isBomb) {
          this.allTileCovers(false)
          alert('Boom');
          return;
        } else if (this.board[row][col].solution==0) {
          this.openSurroundingCell(row, col)
        }
        if (event) {
          this.checkWon();
        }
      }
    },
    flagCell(row, col, event) {
      this.board[row][col].isFlagged = !this.board[row][col].isFlagged;
      var target=null
      if (event) {
        target = event.currentTarget
      } else {
        target = document.getElementById(row+'-'+col);
      }
      if (this.board[row][col].isFlagged) {
        target.querySelector('.fa-flag-checkered').style.display='';
      } else {
        target.querySelector('.fa-flag-checkered').style.display='none';
      }
      this.checkWon();
    },
    clearCell(row, col) {
      if (! (this.board[row][col].isClosed||this.board[row][col].isBomb)) {
        var answer=0;
        for (var u=row-1; u<=row+1; u++) {
          // up, middle, bottom; relative to current cell
          if (u<0 || u==this.dim) continue;
          for (var lr=col-1; lr<=col+1; lr++) {            
            // left, center, right; relative to current cell
            if (lr<0 || lr==this.dim) continue;
            if (this.board[u][lr].isFlagged) {
              answer++;
            }
          }
        }
        if (this.board[row][col].solution==answer) {
          this.openSurroundingCell(row, col)
        }
        this.checkWon();
      }
    },
  },
  created() {
    this.resetBoard();
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.minefield {
  display: flex;
  justify-content: center;
}
.cell {
  border: 1px solid darkgray;
  width: 1.4em;
  height: 1.4em;
}
.cover {
  position: absolute;
  font-size:0.8em; 
  margin-top: -1.9em; 
  margin-left: -1.2em;
}
.reveal {
  display: none;
}
</style>
