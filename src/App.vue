<template>
  <div class="gomoku-container" :style="{ backgroundColor: bgColor }">
    <button class="new-game-button" @click="reset">开始新游戏</button>
    <canvas ref="canvas" class="gomoku-canvas"></canvas>
  </div>
</template>
<script>
export default {
  data() {
    return {
      cellWidth: 40, // 棋盘单元格宽度
      canvasWidth: 600, // 画布宽度
      canvasHeight: 600, // 画布高度
      pieces: [], // 存储已经下的棋子 {x, y, color}
      currentPlayer: 'white', // 当前下棋玩家，白色先行
      isGameOver: false, // 游戏是否结束
      winner: null, // 获胜者
    
    }
  },
  computed: {
    cellNum() {
      return this.canvasWidth / this.cellWidth
    }
  },
  mounted() {
    this.canvas = this.$refs.canvas
    this.ctx = this.canvas.getContext('2d')
    this.canvas.width = this.canvasWidth
    this.canvas.height = this.canvasHeight
    this.canvas.addEventListener('click', this.onClick)

    this.drawBoard()
  },
  methods: {
    onClick(event) {
      if (this.isGameOver) {
        return
      }

      const x = Math.round((event.offsetX - this.cellWidth / 2) / this.cellWidth)
      const y = Math.round((event.offsetY - this.cellWidth / 2) / this.cellWidth)
      if (this.canPlacePiece(x, y)) {
        this.placePiece(x, y, this.currentPlayer)
        this.checkWin(x, y)
        if (!this.isGameOver) {
          this.currentPlayer = this.currentPlayer === 'white' ? 'black' : 'white'
        }
      }
    },
    drawBoard() {
      this.ctx.fillStyle = '#EEEED1'
      this.ctx.fillRect(0, 0, this.canvasWidth, this.canvasHeight)

      this.ctx.strokeStyle = 'black'
      this.ctx.lineWidth = 3

      for (let i = 0; i < this.cellNum; i++) {
        this.ctx.beginPath()
        this.ctx.moveTo((i + 0.5) * this.cellWidth, this.cellWidth / 2)
        this.ctx.lineTo((i + 0.5) * this.cellWidth, this.canvasHeight - this.cellWidth / 2)
        this.ctx.stroke()

        this.ctx.beginPath()
        this.ctx.moveTo(this.cellWidth / 2, (i + 0.5) * this.cellWidth)
        this.ctx.lineTo(this.canvasWidth - this.cellWidth / 2, (i + 0.5) * this.cellWidth)
        this.ctx.stroke()
      }
    },
    canPlacePiece(x, y) {
      return !this.pieces.some(p => p.x === x && p.y === y)
    },
    placePiece(x, y, color) {
      const radius = this.cellWidth / 2 - 4
      const cx = (x + 0.5) * this.cellWidth
      const cy = (y + 0.5) * this.cellWidth

      this.ctx.beginPath()
      this.ctx.arc(cx, cy, radius, 0, 2 * Math.PI)
      this.ctx.fillStyle = color
      this.ctx.fill()

      this.pieces.push({ x, y, color })
    },
    checkWin(x, y) {
      const lines = [
        // 横线
        [
          [x - 4, y], [x - 3, y], [x - 2, y], [x - 1, y], [x, y], [x + 1, y], [x + 2, y], [x + 3, y], [x + 4, y]
        ],
        // 竖线
        [
          [x, y - 4], [x, y - 3], [x, y - 2], [x, y - 1], [x, y], [x, y + 1], [x, y + 2], [x, y + 3], [x, y + 4]
        ],
        // 右上到左下斜线
        [
          [x - 4, y - 4], [x - 3, y - 3], [x - 2, y - 2], [x - 1, y - 1], [x, y], [x + 1, y + 1], [x + 2, y + 2], [x + 3, y + 3], [x + 4, y + 4]
        ],
        // 左上到右下斜线
        [
          [x + 4, y - 4], [x + 3, y - 3], [x + 2, y - 2], [x + 1, y - 1], [x, y], [x - 1, y + 1], [x - 2, y + 2], [x - 3, y + 3], [x - 4, y + 4],
        ]
      ]

      for (const line of lines) {
        let count = 0
        for (const [lx, ly] of line) {
          const piece = this.pieces.find(p => p.x === lx && p.y === ly)
          if (piece && piece.color === this.currentPlayer) {
            count++
          } else {
            count = 0
          }

          if (count === 5) {
            this.isGameOver = true
            this.winner = this.currentPlayer
            alert(`${this.winner} wins!`)
            return
          }
        }
      }

      // 判断平局，如果所有点都下了就是平局
      if (this.pieces.length === this.cellNum * this.cellNum) {
        this.isGameOver = true
        alert('平局')
      }
    },
    reset() {
      this.pieces = []
      this.currentPlayer = 'white'
      this.isGameOver = false
      this.winner = null
      this.drawBoard()
    }
  }
}
</script>
<style scoped>
.gomoku-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

.gomoku-canvas {
  border: 1px solid black;
}

.new-game-button {
  font-size: 20px;
  padding: 10px;
  width: 200px;
  margin-bottom: 20px;
}
</style>
