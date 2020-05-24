<template>
    <div>
        <div class="cell">
        <i class="fas fa-bomb" v-if="isBomb"></i>
        <i v-else-if="solution > 0" v-on:dblclick="clearCell">{{solution}}</i>
        <i class="fas fa-stop" style="color:white;" v-else></i>
        </div>
        <span class="fa-stack fa-2x cover" ref="cover"
        v-on:click="openCell"
        v-on:click.right="flagCell"
        @contextmenu.prevent>
        <i class="fas fa-stop fa-stack-2x"></i>
        <i class="fas fa-flag-checkered fa-stack-1x fa-inverse" ref="flag" v-show="isFlagged"></i>
        </span>
    </div>
</template>

<script>
    import eventBus from './EventBus'

    export default {
        name: "tile",
        props: {
            density: {
                type: Number,
                default: 0.1
            },
            rowidx: {
                type: Number,
                default: 0
            },
            colidx: {
                type: Number,
                default: 0
            }
        },
        data() {
            return {
                solution: 0,
                isClosed: true,
                isFlagged: false,
                isBomb: false,
                guessing: 0
            }
        },
        methods: {
            openCell(event) {
                var target = null;
                if (event) {
                    target = event.currentTarget
                } else {
                    target = this.$refs.cover;
                }
                if (!this.isFlagged) {
                    this.isClosed = false;
                    target.classList.add('reveal')
                    eventBus.$emit('opened', {row: this.rowidx, col: this.colidx})
                    if (this.isBomb) {
                        eventBus.$emit('broadcast', 'boom')
                        //alert('Boom');
                        return;
                    } else if (this.solution==0) {
                        eventBus.$emit('iamopen', {row: this.rowidx, col: this.colidx})
                    }
                }
            },
            myConcernedEvent(payload) {
                if (payload.row>=this.rowidx-1 && payload.row<=this.rowidx+1 &&
                    payload.col>=this.colidx-1 && payload.col<=this.colidx+1) {
                        return true;
                }
                return false;
            },
            neighborOpened(payload) {
                if (this.myConcernedEvent(payload) &&
                    this.isClosed && !this.isFlagged) {
                    this.openCell(null);
                }
            },
            neighboorIsBomb(payload) {
                if (this.myConcernedEvent(payload)) {
                    this.solution++;
                }
            },
            neighboorIsFlagged(payload) {
                if (this.myConcernedEvent(payload)) {
                    this.guessing++;
                }
            },
            flagCell(event) {
                this.isFlagged = !this.isFlagged;
                var target=null
                if (event) {
                    target = event.currentTarget
                } else {
                    target = this.$refs.cover
                }
                if (this.isFlagged) {
                    target.querySelector('.fa-flag-checkered').style.display='';
                } else {
                    target.querySelector('.fa-flag-checkered').style.display='none';
                }
                eventBus.$emit('iamaflag', {row: this.rowidx, col: this.colidx})
            },
            clearCell() {
                if (this.guessing==this.solution) {
                    eventBus.$emit('iamopen', {row: this.rowidx, col: this.colidx})
                }
            },
            processBroadcast(message) {
                // console.log(`processBroadcast ${message} in ${this.rowidx} ${this.colidx}`)
                if (message=='iamabomb') {
                    if (this.isBomb) {
                        eventBus.$emit('iamabomb', {row: this.rowidx, col: this.colidx})
                    }
                } else if (message=='reset'){
                    if (this.$refs.cover && this.$refs.cover.classList.contains("reveal")) {
                        this.$refs.cover.classList.remove("reveal")
                    }
                    if (this.$refs.flag) {
                        this.$refs.flag.style.display='none'
                    }
                    this.solution=0
                    this.isClosed=true
                    this.isFlagged=false
                    this.isBomb=(Math.random()<this.density);
                    this.guessing=0
                } else if (message=='boom') {
                    if (!this.$refs.cover.classList.contains("reveal")) {
                        this.$refs.cover.classList.add("reveal")
                    }
                }
            }
        },
        created() {
            eventBus.$on('iamopen', this.neighborOpened)
            eventBus.$on('iamabomb', this.neighboorIsBomb)
            eventBus.$on('iamaflag', this.neighboorIsFlagged)
            eventBus.$on('broadcast', this.processBroadcast)
        }
    }
</script>

<style scoped>
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