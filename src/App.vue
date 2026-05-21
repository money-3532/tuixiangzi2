<template>
  <div class="game-wrapper">
    <div class="game-container">
      <canvas id="gameCanvas" ref="canvas" width="400" height="400"></canvas>
    </div>
    <div class="info-bar">
      <div class="level-box">📦 第 {{ currentLevel + 1 }} / {{ totalLevels }} 关</div>
      <button @click="resetLevel">🔄 重置关卡</button>
      <div class="message">⭐ 把箱子推到红色星星上 ⭐</div>
    </div>
    <div class="controls">🎮 方向键移动 | R 键重置当前关 | 全部通关会有烟花彩蛋！</div>
  </div>
</template>

<script>
export default {
  name: 'SokobanGame',
  data() {
    return {
      levels: [
        // 第1关
        [
          '########',
          '#      #',
          '#  $   #',
          '#  .   #',
          '#   @  #',
          '#      #',
          '#      #',
          '########',
        ],
        // 第2关 双箱子
        [
          '########',
          '#  .   #',
          '#  $   #',
          '#      #',
          '#  $   #',
          '#  .   #',
          '#    @ #',
          '########',
        ],
        // 第3关 中等难度
        [
          '########',
          '# .$.  #',
          '#  $   #',
          '#  ##  #',
          '#   @  #',
          '#      #',
          '#      #',
          '########',
        ],
        // 第4关 困难挑战
        [
          '########',
          '# .$.  #',
          '# #$   #',
          '# .$   #',
          '#  @   #',
          '#      #',
          '#      #',
          '########',
        ],
      ],
      currentLevel: 0,
      levelMap: [],
      playerX: -1,
      playerY: -1,
      boxCount: 0,
      boxesOnTarget: 0,
      CELL_SIZE: 50,
      canvas: null,
      ctx: null,
    }
  },
  computed: {
    totalLevels() {
      return this.levels.length
    },
  },
  mounted() {
    this.canvas = this.$refs.canvas
    this.ctx = this.canvas.getContext('2d')
    this.loadLevel(0)
    window.addEventListener('keydown', this.handleKeydown)
  },
  beforeUnmount() {
    window.removeEventListener('keydown', this.handleKeydown)
  },
  methods: {
    loadLevel(idx) {
      const raw = this.levels[idx]
      this.levelMap = raw.map((row) => [...row])
      this.playerX = -1
      this.playerY = -1
      this.boxCount = 0
      this.boxesOnTarget = 0

      for (let y = 0; y < 8; y++) {
        for (let x = 0; x < 8; x++) {
          const c = this.levelMap[y][x]
          if (c === '@' || c === '+') {
            this.playerX = x
            this.playerY = y
            this.levelMap[y][x] = ' '
          }
          if (c === '$' || c === '*') this.boxCount++
        }
      }
      this.drawGame()
    },

    drawGame() {
      this.ctx.clearRect(0, 0, 400, 400)
      for (let y = 0; y < 8; y++) {
        for (let x = 0; x < 8; x++) {
          const cell = this.levelMap[y][x]
          const px = x * this.CELL_SIZE
          const py = y * this.CELL_SIZE
          const centerX = px + 25
          const centerY = py + 25

          if (cell === '#') {
            this.ctx.fillStyle = '#5a3a22'
            this.ctx.fillRect(px, py, this.CELL_SIZE, this.CELL_SIZE)
            this.ctx.strokeStyle = '#8b5a3b'
            this.ctx.lineWidth = 3
            this.ctx.strokeRect(px + 3, py + 3, this.CELL_SIZE - 6, this.CELL_SIZE - 6)
          } else if (cell === '.') {
            this.drawStar(centerX, centerY, 14)
          } else if (cell === '$') {
            this.ctx.fillStyle = '#ff9c47'
            this.ctx.fillRect(px + 6, py + 6, this.CELL_SIZE - 12, this.CELL_SIZE - 12)
            this.ctx.strokeStyle = '#a85a1c'
            this.ctx.lineWidth = 3
            this.ctx.strokeRect(px + 6, py + 6, this.CELL_SIZE - 12, this.CELL_SIZE - 12)
          } else if (cell === '*') {
            this.ctx.fillStyle = '#ff6b6b'
            this.ctx.fillRect(px + 6, py + 6, this.CELL_SIZE - 12, this.CELL_SIZE - 12)
            this.ctx.strokeStyle = '#c72e2e'
            this.ctx.lineWidth = 3
            this.ctx.strokeRect(px + 6, py + 6, this.CELL_SIZE - 12, this.CELL_SIZE - 12)
            this.drawStar(centerX, centerY, 14)
          }
        }
      }

      // 绘制玩家
      const px = this.playerX * this.CELL_SIZE + 25
      const py = this.playerY * this.CELL_SIZE + 22
      this.ctx.fillStyle = '#ffcc33'
      this.ctx.beginPath()
      this.ctx.ellipse(px, py - 12, 14, 6, 0, 0, Math.PI * 2)
      this.ctx.fill()
      this.ctx.fillStyle = '#ffddbb'
      this.ctx.beginPath()
      this.ctx.arc(px, py, 14, 0, Math.PI * 2)
      this.ctx.fill()
      this.ctx.fillStyle = '#222'
      this.ctx.beginPath()
      this.ctx.arc(px - 5, py - 2, 2.5, 0, Math.PI * 2)
      this.ctx.arc(px + 5, py - 2, 2.5, 0, Math.PI * 2)
      this.ctx.fill()
      this.ctx.fillStyle = '#4287f5'
      this.ctx.fillRect(px - 10, py + 12, 20, 14)
    },

    drawStar(cx, cy, r) {
      this.ctx.fillStyle = '#ff2222'
      this.ctx.beginPath()
      for (let i = 0; i < 5; i++) {
        let angle = Math.PI / 2 + (i * (Math.PI * 2)) / 5
        let x = cx + Math.cos(angle) * r
        let y = cy - Math.sin(angle) * r
        if (i === 0) this.ctx.moveTo(x, y)
        else this.ctx.lineTo(x, y)
        angle += Math.PI / 5
        x = cx + Math.cos(angle) * (r * 0.4)
        y = cy - Math.sin(angle) * (r * 0.4)
        this.ctx.lineTo(x, y)
      }
      this.ctx.closePath()
      this.ctx.fill()
    },

    move(dx, dy) {
      const nx = this.playerX + dx
      const ny = this.playerY + dy
      if (this.levelMap[ny][nx] === '#') return

      const cell = this.levelMap[ny][nx]
      // 碰到箱子或已经在目标上的箱子需要处理
      if (cell === '$' || cell === '*') {
        const nnx = nx + dx
        const nny = ny + dy
        if (nny < 0 || nny >= 8 || nnx < 0 || nnx >= 8) return
        const targetCell = this.levelMap[nny][nnx]
        if (targetCell === ' ' || targetCell === '.') {
          // 推箱子
          const wasOnTarget = cell === '*'
          const willBeOnTarget = targetCell === '.'

          // 更新箱子位置
          this.levelMap[ny][nx] = wasOnTarget ? '.' : ' '
          this.levelMap[nny][nnx] = willBeOnTarget ? '*' : '$'

          if (wasOnTarget) this.boxesOnTarget--
          if (willBeOnTarget) this.boxesOnTarget++

          // 移动玩家
          this.playerX = nx
          this.playerY = ny
          this.drawGame()
          this.checkWin()
        }
        return
      }

      // 普通移动
      if (cell === ' ' || cell === '.') {
        this.playerX = nx
        this.playerY = ny
        this.drawGame()
      }
    },

    checkWin() {
      if (this.boxesOnTarget === this.boxCount) {
        setTimeout(() => {
          if (this.currentLevel < this.totalLevels - 1) {
            this.currentLevel++
            this.loadLevel(this.currentLevel)
            alert('🎉 太厉害了！进入下一关！')
          } else {
            alert('🏆 恭喜通关全部4关！你真棒！')
            this.fireworks()
          }
        }, 200)
      }
    },

    resetLevel() {
      this.loadLevel(this.currentLevel)
    },

    handleKeydown(e) {
      const key = e.key
      switch (key) {
        case 'ArrowUp':
          e.preventDefault()
          this.move(0, -1)
          break
        case 'ArrowDown':
          e.preventDefault()
          this.move(0, 1)
          break
        case 'ArrowLeft':
          e.preventDefault()
          this.move(-1, 0)
          break
        case 'ArrowRight':
          e.preventDefault()
          this.move(1, 0)
          break
        case 'r':
        case 'R':
          e.preventDefault()
          this.resetLevel()
          break
        default:
          break
      }
    },

    fireworks() {
      const count = 120
      for (let i = 0; i < count; i++) {
        const circle = document.createElement('div')
        document.body.appendChild(circle)
        const angle = Math.random() * Math.PI * 2
        const distance = 120 + Math.random() * 240
        const dx = Math.cos(angle) * distance
        const dy = Math.sin(angle) * distance
        circle.style.cssText = `
          position: fixed;
          width: 8px; height: 8px;
          background: hsl(${Math.random() * 360}, 100%, 60%);
          border-radius: 50%;
          left: 50%; top: 50%;
          pointer-events: none;
          transform: translate(-50%, -50%);
          animation: firework 1.4s ease-out forwards;
        `
        circle.style.setProperty('--dx', dx + 'px')
        circle.style.setProperty('--dy', dy + 'px')
        setTimeout(() => circle.remove(), 1400)
      }
    },
  },
}
</script>

<style scoped>
* {
  user-select: none;
}
.game-wrapper {
  background: #d9b382;
  padding: 25px 30px 20px 30px;
  border-radius: 65px;
  box-shadow:
    0 20px 0 #7a4c2c,
    inset 0 1px 4px rgba(255, 255, 200, 0.8);
  display: inline-block;
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
  margin-top: 12px;
  text-align: center;
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

/* 烟花动画 */
@keyframes firework {
  0% {
    transform: translate(-50%, -50%);
    opacity: 1;
  }
  100% {
    transform: translate(calc(-50% + var(--dx)), calc(-50% + var(--dy)));
    opacity: 0;
  }
}
</style>
