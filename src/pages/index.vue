<script setup lang="ts">
interface BlockState {
  x: number
  y: number
  revealed?: boolean //翻开
  mine?: boolean //是否炸弹
  flagged?: boolean //旗帜
  adjacentMines: number //周围炸弹数量
}

const WIDTH = 10
const HEIGHT = 10
const state = reactive(Array.from({length: HEIGHT}, (_, y) =>
    Array.from({length: WIDTH},
        (_, x): BlockState => ({
          x,
          y,
          adjacentMines: 0,
          revealed: false
        })
    )
))

function generateMines(initial:BlockState) {
  for (const row of state) {
    for (const block of row) {
      if(Math.abs(initial.x-block.x)<1&&Math.abs(initial.y-block.y)<2){
        console.log(initial.x,block.x)
        continue
      }
      if(Math.abs(initial.y-block.y)<1&&Math.abs(initial.x-block.x)<2){
        console.log(initial.y,block.y)
        continue
      }
      block.mine = Math.random() < 0.2
    }
  }

  updateNumbers()
}

const directions = [
  [0, 1],
  [1, 1],
  [1, 0],
  [-1, 1],
  [1, -1],
  [0, -1],
  [-1, 0],
  [-1, -1]
]

function updateNumbers() {
  state.forEach((raw, y) => {
    raw.forEach((block, x) => {
      if (block.mine) {
        return
      }
      directions.forEach(([dx, dy]) => {
        const x2 = x + dx
        const y2 = y + dy
        if (x2 < 0 || y2 < 0 || x2 >= WIDTH || y2 >= HEIGHT)
          return
        if (state[y2][x2].mine)
          block.adjacentMines++
      })
    })
  })
}
//第一次点击
let mineGenerated=false
const dev=true
function OnClick(block: BlockState) {
  if(!mineGenerated){
    generateMines(block)
    mineGenerated=true
  }

  if (!block.revealed) {
    block.revealed = true
  }
  if(block.mine){
    alert("BOOOOM!!!")
  }
}

const numberColors = [
  'text-transparent',
  'text-blue-500',
  'text-green-500',
  'text-yellow-500',
  'text-orange-500',
  'text-red-500',
  'text-purple-500',
  'text-pink-500',
]

function getBlockClass(block: BlockState) {
  if(!block.revealed){
    return 'bg-gray-500/10'
  }
  return block.mine ?
      'text-red'
      : numberColors[block.adjacentMines]
}
function expendZero(block:BlockState){
  if(){

  }
}
</script>

<template>
  <div>
    Minesweeper
    <div v-for="(row,y) in state"
         :key="y"
         flex="~"
         items-center justify-center

    >
      <button
          v-for="(item,x) in row"
          :key="x"
          flex="~"
          items-center justify-center m="0.5"
          hover="bg-gray/10"
          w-8
          h-8
          border="1 gray-400/10"
          :class="getBlockClass(item)"
          @click="OnClick(item)">
          <block v-if="item.revealed">
            <div v-if="item.mine" i-mdi-mine/>
            <div v-else>
              {{ item.adjacentMines }}
            </div>
          </block>

      </button>
    </div>
  </div>
</template>
