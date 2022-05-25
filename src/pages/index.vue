<script setup lang="ts">
interface BlockState {
  x: number
  y: number
  revealed?: boolean //翻开
  mine?: boolean //是否炸弹
  flagged?: boolean //旗帜
  adjacentMines: number //周围炸弹数量
}

const WIDTH = 3
const HEIGHT = 3
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
//生成炸弹
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
      block.mine = Math.random() < 0.1
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
      if (block.mine)
        return
      getSiblings(block).forEach((a)=>{
        if(a?.mine){
          block.adjacentMines++
        }
      })

    })
  })
}
//第一次点击
let mineGenerated=false


function OnClick(block: BlockState) {
  if(block.flagged){
    return
  }
  if(!mineGenerated){
    generateMines(block)
    mineGenerated=true
  }

  if (!block.revealed) {
    block.revealed = true
  }
  if(block.mine){
    for (const row of state) {
      for (const b of row) {
        if(b.mine){
          b.revealed=true
        }
      }
    }
    alert("BOOOOM!!!")
    return;
  }
  checkGamesState()
  expendZero(block)
}
function OnClickRight(block:BlockState) {
  if(!block.revealed){
    block.flagged=!block.flagged
  }

  checkGamesState()
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
  if(block.flagged){
    return 'bg-gray-500/10'
  }
  if(!block.revealed){
    return 'bg-gray-500/10 hover:bg-gray/10'
  }
  return block.mine ?
      'text-red'
      : numberColors[block.adjacentMines]
}
//只有翻开为0的格子触发连锁
function expendZero(block:BlockState){
  //如果有数字 ||已经翻开 不做处理
  if(block.adjacentMines){
    return
  }

  getSiblings(block).forEach((b)=>{
    if( !b.revealed&&!b.flagged){
      b.revealed=true
      expendZero(b)
    }
  })
}
//计算周围8个格子
function getSiblings(block:BlockState) {
  return directions.map(([dx, dy]) => {
    const x2 = block.x + dx
    const y2 = block.y + dy
    if (x2 < 0 || y2 < 0 || x2 >= WIDTH || y2 >= HEIGHT)
      return undefined
    return state[y2][x2]
  })
  .filter(Boolean) as BlockState[]
}
function checkGamesState(){
  if(!mineGenerated){
    return
  }
  const temp=state.flat()
  if(temp.every((item)=>item.revealed||item.flagged)){
    if(temp.some((item)=>item.flagged&&!item.mine)){
      alert("You cheat!")
    }
    else {
      alert("You win!")
      console.log(Object.assign({}, state))
    }
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
          w-8
          h-8
          border="1 gray-400/10"
          :class="getBlockClass(item)"
          @click="OnClick(item)"
          @contextmenu.prevent="OnClickRight(item)"
      >
          <template v-if="item.flagged">
            <div i-mdi-flag></div>
          </template>
          <template v-if="item.revealed">
            <div v-if="item.mine" i-mdi-mine/>
            <div v-else>
              {{ item.adjacentMines }}
            </div>
          </template>
      </button>
    </div>
  </div>
</template>
