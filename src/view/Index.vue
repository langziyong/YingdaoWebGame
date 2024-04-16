<script setup lang="ts">

import {computed, onMounted, ref} from "vue";

interface GameFrame {
  x: number
  y: number
  width: number
  height: number
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
  nowFrame: GameFrame = {x: 0, y: 0, width: 0, height: 0}
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
    let frame: GameFrame = {x: 0, y: 0, width: 0, height: 0}
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
    this.clockTime = 0
    this.avgTime = 0
    clearInterval(this.intervalId)
  }

  start() {
    this.startClock()
    this.generateFrame()
    this.status = GameStatus.PROCESSING
  }

  stop() {
    this.stopClock()
    this.score = 0
    this.status = GameStatus.READY
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

function testAddCssClass() {
  var styleElement = document.createElement('style');
  styleElement.innerHTML = '.highlighted { display: none; }';
  document.head.appendChild(styleElement);
}

const cheat = ref(false)
const cheatCode = ref<Array<string>>([])
const allowKey = new Set(['x', 'i'])
onMounted(() => {
  rootEl = document.querySelector(".root")
  document.addEventListener('keydown', function (event) {
    if (event.code === 'Space') {
      event.preventDefault()
      if (gameInstance.value.status === GameStatus.READY) {
        gameInstance.value.start()
      } else if (gameInstance.value.status === GameStatus.PROCESSING) {
        gameInstance.value.stop()
      }
    }
  })

  document.addEventListener('keydown', function (event) {
    if (event.ctrlKey && allowKey.has(event.key)) {
      event.preventDefault()
      cheatCode.value.push(event.key)
    }
    if (cheatCode.value.length === 4) {
      let code = cheatCode.value.join('')
      cheatCode.value = []
      if (code === "xixi") {
        cheat.value = true
        console.log("嘻嘻")
      }
    }
  })

  document.addEventListener('keydown', function (event) {
    if (event.key === 'x' && cheat.value) {
      event.preventDefault()
      gameInstance.value.clickHandler()
    }
  })
})


</script>

<template>
  <div class="root">
    <div class="start" v-if="gameInstance.status===GameStatus.READY">
      <div>按下空格随时暂停游戏</div>
      <div class="btn" @click="gameInstance.start">开始游戏</div>
      <div class="btn highlighted" @click="testAddCssClass">生成样式类</div>
    </div>
    <div class="status" v-else>
      <div class="score">{{ gameInstance.score }}</div>
      <div class="avg">{{ gameInstance.avgTime.toFixed(2) }}ms</div>
      <div class="avg clock">{{ (gameInstance.clockTime / 1000).toFixed(2) }}s</div>
    </div>
    <div class="block_pool" v-if="gameInstance.status===GameStatus.PROCESSING">
      <div class="block"
           :style="style" @click="gameInstance.clickHandler"/>
    </div>

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