<script setup lang="ts">

interface BlockState {
  x: number
  y: number
  // 是否翻开
  revealed: boolean
  // 是否是炸弹
  mine?: boolean
  // 是否被标记
  flagged?: boolean
  adjacentMine: number
}

const WIDTH = 10
const HEIGHT = 10

const state = reactive(
  Array.from({ length: HEIGHT }, (_, y) =>
    Array.from({ length: WIDTH },
      (_, x): BlockState => ({
        x,
        y,
        adjacentMine: 0,
        revealed: false,
      }),
    ),
  ),
)

const cheatConfig = reactive({
  cheat: false,
})

// 生成炸弹
function generateMines(initial: BlockState) {
  for (const row of state) {
    for (const block of row) {
      if (Math.abs(initial.x - block.x) + Math.abs(initial.y - block.y) <= 2)
        continue
      block.mine = Math.random() < 0.1
    }
  }
  updateNumbers()
}

const directions = [
  [-1, -1],
  [-1, 0],
  [-1, 1],
  [0, -1],
  [0, 1],
  [1, -1],
  [1, 0],
  [1, 1],
]

// 不同数字的不同颜色
const numberColors = [
  'text-transparent',
  'text-blue-500',
  'text-green-500',
  'text-red-500',
  'text-yellow-500',
  'text-purple-500',
  'text-orange-500',
  'text-indigo-500',
]

function getSiblings(block: BlockState) {
  return directions.map(([dx, dy]) => {
    const x2 = block.x + dx
    const y2 = block.y + dy
    if (x2 < 0 || x2 >= WIDTH || y2 < 0 || y2 >= HEIGHT) return undefined
    return state[y2][x2]
  }).filter(Boolean) as BlockState[]
}


function checkGameState() {
  if (!mineGenerated) return
  const blocks = state.flat()

  if (blocks.every(block => block.revealed || block.mine)) {
    // if (blocks.some(block => !block.mine && block.flagged)) {
    //   alert('Game Over')
    // } else {
      alert('胜利！！！！')
    // }
  }
}

// 计算周围炸弹个数
function updateNumbers() {
  state.forEach((raw, y) => {
    raw.forEach((block, x) => {
      if (block.mine) return
      getSiblings(block).forEach(b => {
        if (b.mine) block.adjacentMine++
      })
    })
  })
}

// 展开所有空白区域
function expandZero(block: BlockState) {
  if (block.adjacentMine) return

  getSiblings(block).forEach(s => {
    if (!s.revealed) {
      s.revealed = true
      expandZero(s)
    }
  })
}

let mineGenerated = false

function onRightClick(block: BlockState) {
  if (block.revealed) {
    return
  }
  block.flagged = !block.flagged
  checkGameState()
}

function onClick(block: BlockState) {
  if (block.flagged || block.revealed) {
    return
  }
  if (!mineGenerated) {
    generateMines(block)
    mineGenerated = true
  }
  
  block.revealed = true
  if (block.mine) {
    alert('BOOOOOM!')
    return
  }
  expandZero(block)
  checkGameState()
}

function getBlockClass(block: BlockState) {
  if (!block.revealed && !cheatConfig.cheat) return 'bg-gray-500/10 hover:bg-gray-500/20'
  return block.mine ? 'bg-red-500/50' : numberColors[block.adjacentMine]
}

function reset() {
  state.forEach(row => row.forEach(block => {
    block.revealed = false
    block.flagged = false
    block.mine = false
    block.adjacentMine = 0
  }))
  mineGenerated = false
}

function showMine() {
  cheatConfig.cheat = true
  setTimeout(() => {
    cheatConfig.cheat = false
  }, 1500)
}
</script>

<template>
  <div>
    扫雷
    <div p5>
      <div
        v-for="(row, y) in state"
        :key="y"
        flex="~"
        items-center justify-center
      >
        <button
          v-for="(block,x) in row"
          :key="x"
          flex="~"
          items-center justify-center
          w-10 h-10 m="0.5"
          border="1 gray-400/10"
          :class="getBlockClass(block)"
          @click="onClick(block)"
          @contextmenu.prevent="onRightClick(block)"
        >
          <template v-if="block.flagged">
            <div i-mdi-flag text-red />
          </template>
          <template v-else-if="block.revealed || cheatConfig.cheat">
            <div v-if="block.mine" i-mdi-mine></div>
            <div v-else>{{ block.adjacentMine || '-' }}</div>
          </template>
        </button>
      </div>
      <button mt-10 @click="showMine()">偷看一下</button>
      <button mt-10 ml-10 @click="reset()">重新开始</button>
    </div>
  </div>
</template>
