<script setup lang="ts">

import {computed, onMounted, ref} from "vue";

interface GameFrame {
  x: number
  y: number
}

let rootEl: Element | null
const rootSize = computed(() => {
  if (rootEl)
    return {
      width: rootEl.clientWidth,
      height: rootEl.clientHeight
    }
  else
    return null
})

enum GameStatus {
  READY = 0,
  PROCESSING = 1,
  OVER = 2,
  ERROR = -1
}

class Game {
  nowFrame: GameFrame = {x: 0, y: 0}
  score: number = 0
  startTime: Date | null = null
  avgTime: number = 0
  clockTime: number = 0
  intervalId: number = -1
  status: GameStatus = GameStatus.READY

  constructor() {

  }

  generateFrame(): void {
    if (!rootSize.value) return
    let frame: GameFrame = {x: 0, y: 0}
    frame.x = Math.floor(Math.random() * (rootSize.value.width - 200))
    frame.y = Math.floor(Math.random() * (rootSize.value.height - 200))
    this.nowFrame = frame
  }

  clickHandler(): void {
    this.score++
    if (!this.startTime) return
    this.avgTime = (new Date().getTime() - this.startTime.getTime()) / this.score
    this.generateFrame()
  }

  startClock(): void {
    this.startTime = new Date()
    this.intervalId = setInterval(() => {
      if (!this.startTime) {
        this.stopClock()
        return
      }
      this.clockTime = new Date().getTime() - this.startTime?.getTime()
    }, 20)
  }

  stopClock() {
    this.startTime = null
    clearInterval(this.intervalId)
  }

  start() {
    this.startClock()
    this.generateFrame()
    this.status = GameStatus.PROCESSING
  }
  
}

const gameInstance = ref<Game>(new Game())
const style = computed(() => {
  if (!gameInstance.value) return null
  return {
    top: gameInstance.value.nowFrame.y + "px",
    left: gameInstance.value.nowFrame.x + "px"
  }
})


onMounted(() => {
  rootEl = document.querySelector(".root")
})


</script>

<template>
  <div class="root">
    <div class="start" v-if="gameInstance.status===GameStatus.READY">
      <div class="btn" @click="gameInstance.start">开始游戏</div>
    </div>
    <div class="status" v-else>
      <div class="score">{{ gameInstance.score }}</div>
      <div class="avg">{{ gameInstance.avgTime.toFixed(2) }}ms</div>
      <div class="avg clock">{{ (gameInstance.clockTime / 1000).toFixed(2) }}s</div>
    </div>
    <div class="block" v-if="gameInstance.status===GameStatus.PROCESSING"
         :style="style" @click="gameInstance.clickHandler"/>
  </div>
</template>

<style lang="less" scoped>
.root {
  position: relative;
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;


  .status {
    width: 30%;
    font-size: 5rem;
    text-align: center;

    .avg {
      font-size: 2rem;
    }
  }
}

.block {
  content: '';
  position: absolute;
  width: 200px;
  height: 200px;
  background: aquamarine;
  opacity: 0.5;
}

</style>