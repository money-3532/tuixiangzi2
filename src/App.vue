<template>
  <div class="game-page">
    <div class="game-wrapper">
      <div class="game-container">
        <canvas ref="canvas" width="400" height="400"></canvas>
      </div>

      <div class="info-bar">
        <div class="level-box">
          📦 第 <span>{{ currentLevel + 1 }}</span> / <span>{{ levels.length }}</span> 关
        </div>
        <button @click="resetLevel">🔄 重置关卡</button>
        <div class="message">⭐ 把箱子推到红色星星上 ⭐</div>
      </div>

      <div class="controls">🎮 方向键移动 | R 键重置当前关 | 全部通关会有烟花彩蛋！</div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

// 6 关地图
const levels = [
  ['########', '#      #', '#  $   #', '#  .   #', '#  @   #', '#      #', '#      #', '########'],
  ['########', '#      #', '#  $   #', '#  .   #', '#   @  #', '#      #', '#      #', '########'],
  ['########', '#  .   #', '#  $   #', '#      #', '#  $   #', '#  .   #', '#    @ #', '########'],
  ['########', '# .$.  #', '#  $   #', '#  ##  #', '#   @  #', '#      #', '#      #', '########'],
  ['########', '# .$.  #', '# #$   #', '# .$   #', '#  @   #', '#      #', '#      #', '########'],
  ['########', '#.$. $.#', '# #$#  #', '# .$ . #', '#  @   #', '#      #', '#      #', '########'],
]

const canvas = ref(null)
const currentLevel = ref(0)
let levelMap = []
let playerX = -1
let playerY = -1
let boxCount = 0
let boxesOnTarget = 0
const CELL_SIZE = 50
let ctx = null

onMounted(() => {
  ctx = canvas.value.getContext('2d')
  loadLevel(currentLevel.value)
  window.addEventListener('keydown', onKeyDown)
})

onUnmounted(() => {
  window.removeEventListener('keydown', onKeyDown)
})

// 绘制五角星
function drawStar(cx, cy, r) {
  ctx.fillStyle = '#ff2222'
  ctx.beginPath()
  for (let i = 0; i < 5; i++) {
    let angle = Math.PI / 2 + (i * Math.PI * 2) / 5
    let x = cx + Math.cos(angle) * r
    let y = cy - Math.sin(angle) * r
    if (i === 0) ctx.moveTo(x, y)
    else ctx.lineTo(x, y)
    angle += Math.PI / 5
    x = cx + Math.cos(angle) * (r * 0.4)
    y = cy - Math.sin(angle) * (r * 0.4)
    ctx.lineTo(x, y)
  }
  ctx.closePath()
  ctx.fill()
}

// 加载关卡
function loadLevel(idx) {
  const raw = levels[idx]
  levelMap = raw.map((row) => [...row])
  playerX = -1
  playerY = -1
  boxCount = 0
  boxesOnTarget = 0

  for (let y = 0; y < 8; y++) {
    for (let x = 0; x < 8; x++) {
      const c = levelMap[y][x]
      if (c === '@' || c === '+') {
        playerX = x
        playerY = y
        levelMap[y][x] = ' '
      }
      if (c === '$' || c === '*') boxCount++
    }
  }
  drawGame()
}

// 绘制游戏
function drawGame() {
  ctx.clearRect(0, 0, 400, 400)

  for (let y = 0; y < 8; y++) {
    for (let x = 0; x < 8; x++) {
      const cell = levelMap[y][x]
      const px = x * CELL_SIZE
      const py = y * CELL_SIZE
      const centerX = px + 25
      const centerY = py + 25

      if (cell === '#') {
        ctx.fillStyle = '#5a3a22'
        ctx.fillRect(px, py, CELL_SIZE, CELL_SIZE)
        ctx.strokeStyle = '#8b5a3b'
        ctx.lineWidth = 3
        ctx.strokeRect(px + 3, py + 3, CELL_SIZE - 6, CELL_SIZE - 6)
      } else if (cell === '.') {
        drawStar(centerX, centerY, 14)
      } else if (cell === '$') {
        ctx.fillStyle = '#ff9c47'
        ctx.fillRect(px + 6, py + 6, CELL_SIZE - 12, CELL_SIZE - 12)
        ctx.strokeStyle = '#a85a1c'
        ctx.lineWidth = 3
        ctx.strokeRect(px + 6, py + 6, CELL_SIZE - 12, CELL_SIZE - 12)
      } else if (cell === '*') {
        ctx.fillStyle = '#ff6b6b'
        ctx.fillRect(px + 6, py + 6, CELL_SIZE - 12, CELL_SIZE - 12)
        ctx.strokeStyle = '#c72e2e'
        ctx.lineWidth = 3
        ctx.strokeRect(px + 6, py + 6, CELL_SIZE - 12, CELL_SIZE - 12)
        drawStar(centerX, centerY, 14)
      }
    }
  }

  // 绘制小人
  const px = playerX * CELL_SIZE + 25
  const py = playerY * CELL_SIZE + 22
  ctx.fillStyle = '#ffcc33'
  ctx.beginPath()
  ctx.ellipse(px, py - 12, 14, 6, 0, 0, Math.PI * 2)
  ctx.fill()

  ctx.fillStyle = '#ffddbb'
  ctx.beginPath()
  ctx.arc(px, py, 14, 0, Math.PI * 2)
  ctx.fill()

  ctx.fillStyle = '#222'
  ctx.beginPath()
  ctx.arc(px - 5, py - 2, 2.5, 0, Math.PI * 2)
  ctx.arc(px + 5, py - 2, 2.5, 0, Math.PI * 2)
  ctx.fill()

  ctx.fillStyle = '#4287f5'
  ctx.fillRect(px - 10, py + 12, 20, 14)
}

// 移动
function move(dx, dy) {
  const nx = playerX + dx
  const ny = playerY + dy
  if (levelMap[ny][nx] === '#' || levelMap[ny][nx] === '$' || levelMap[ny][nx] === '*') return
  playerX = nx
  playerY = ny
  drawGame()
}

// 推箱子逻辑
function pushBox(dx, dy) {
  const nx = playerX + dx
  const ny = playerY + dy
  if (levelMap[ny][nx] !== '$' && levelMap[ny][nx] !== '*') return

  const nnx = nx + dx
  const nny = ny + dy
  if (levelMap[nny][nnx] !== ' ' && levelMap[nny][nnx] !== '.') return

  const wasOnTarget = levelMap[ny][nx] === '*'
  const willBeOnTarget = levelMap[nny][nnx] === '.'

  levelMap[ny][nx] = wasOnTarget ? '.' : ' '
  levelMap[nny][nnx] = willBeOnTarget ? '*' : '$'

  if (wasOnTarget) boxesOnTarget--
  if (willBeOnTarget) boxesOnTarget++

  playerX = nx
  playerY = ny
  checkWin()
  drawGame()
}

// 胜利判断
function checkWin() {
  if (boxesOnTarget === boxCount) {
    setTimeout(() => {
      if (currentLevel.value < levels.length - 1) {
        currentLevel.value++
        loadLevel(currentLevel.value)
        alert('🎉 太厉害了！下一关更有挑战！')
      } else {
        alert('🏆 终极通关！你是推箱子大师！')
        fireworks()
      }
    }, 200)
  }
}

// 键盘事件
function onKeyDown(e) {
  if (e.key === 'r' || e.key === 'R') {
    resetLevel()
    return
  }

  let dx = 0,
    dy = 0
  switch (e.key) {
    case 'ArrowUp':
      dy = -1
      break
    case 'ArrowDown':
      dy = 1
      break
    case 'ArrowLeft':
      dx = -1
      break
    case 'ArrowRight':
      dx = 1
      break
    default:
      return
  }

  pushBox(dx, dy)
  move(dx, dy)
}

// 重置
function resetLevel() {
  loadLevel(currentLevel.value)
}

// 烟花
function fireworks() {
  const count = 120
  for (let i = 0; i < count; i++) {
    const circle = document.createElement('div')
    document.body.appendChild(circle)
    circle.style.cssText = `
      position: fixed;
      width: 8px; height: 8px;
      background: hsl(${Math.random() * 360},100%,60%);
      border-radius: 50%;
      left: 50%; top: 50%;
      pointer-events: none;
      transform: translate(-50%,-50%);
      animation: firework 1.4s ease-out forwards;
    `
    const a = Math.random() * Math.PI * 2
    const d = 120 + Math.random() * 240
    circle.style.setProperty('--dx', Math.cos(a) * d + 'px')
    circle.style.setProperty('--dy', Math.sin(a) * d + 'px')
    setTimeout(() => circle.remove(), 1400)
  }
}

// 烟花动画
const style = document.createElement('style')
style.innerHTML = `
@keyframes firework {
  0% { transform: translate(-50%,-50%); opacity:1; }
  100% { transform: translate(calc(-50% + var(--dx)),calc(-50% + var(--dy))); opacity:0; }
}`
document.head.appendChild(style)
</script>

<style scoped>
* {
  user-select: none;
}
.game-page {
  background: linear-gradient(145deg, #1e5a3a 0%, #0f3b24 100%);
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  font-family: 'Segoe UI', 'Comic Neue', 'Courier New', monospace;
  margin: 0;
  padding: 20px;
}
.game-wrapper {
  background: #d9b382;
  padding: 25px 30px 20px 30px;
  border-radius: 65px;
  box-shadow:
    0 20px 0 #7a4c2c,
    inset 0 1px 4px rgba(255, 255, 200, 0.8);
}
.game-container {
  background: #2c1e12;
  padding: 20px;
  border-radius: 35px;
  display: inline-block;
  box-shadow:
    inset 0 0 0 3px #f7e5b5,
    0 10px 20px rgba(0, 0, 0, 0.3);
}
canvas {
  display: block;
  margin: 0 auto;
  border-radius: 20px;
  background-color: #ecd9b4;
  box-shadow: 0 6px 0 #5a3a22;
  cursor: pointer;
}
.info-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 18px;
  gap: 15px;
  flex-wrap: wrap;
}
.level-box {
  background: #2d2218;
  color: #ffdd99;
  padding: 6px 18px;
  border-radius: 60px;
  font-size: 24px;
  font-weight: bold;
  font-family: monospace;
  letter-spacing: 2px;
  box-shadow:
    inset 0 1px 3px #826a48,
    0 3px 0 #2a1a0c;
}
button {
  background: #ffbb77;
  border: none;
  font-size: 20px;
  font-weight: bold;
  padding: 6px 18px;
  border-radius: 50px;
  font-family: inherit;
  cursor: pointer;
  transition: 0.1s linear;
  box-shadow: 0 4px 0 #a5622c;
  color: #2e241a;
}
button:active {
  transform: translateY(2px);
  box-shadow: 0 1px 0 #a5622c;
}
.message {
  background: #000000aa;
  backdrop-filter: blur(8px);
  padding: 6px 14px;
  border-radius: 32px;
  color: #ffe6b3;
  font-weight: bold;
  font-size: 16px;
}
.controls {
  background: #2d241cb3;
  border-radius: 50px;
  padding: 5px 12px;
  color: #fdebb3;
  font-size: 14px;
  text-align: center;
  margin-top: 12px;
}

@media (max-width: 550px) {
  .game-wrapper {
    padding: 15px;
  }
  .level-box {
    font-size: 18px;
  }
  button {
    font-size: 16px;
    padding: 4px 12px;
  }
}
</style>
