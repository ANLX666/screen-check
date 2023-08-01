<template>
  <view class="container" @touchstart="handleTouchStart" @touchmove="handleTouchMove">
    <view v-for="(row, rowIndex) in grid" :key="rowIndex" class="row">
      <view v-for="(col, colIndex) in row" :key="colIndex" class="cell" :style="{
        backgroundColor: col.color,
        width: cellWidth + 'px',
        height: cellHeight + 'px'
      }"></view>
    </view>
  </view>
</template>

<script>
export default {
  name: "lx-screen-check",
  data() {
    return {
      grid: [],  // 格子图
      cellWidth: 0,  // 格子宽度
      cellHeight: 0  // 格子高度
    }
  },
  mounted() {
    this.initGrid()
  },
  methods: {
    // 初始化格子图
    initGrid() {
      const rowSize = 40 // 行数
      const colSize = 12 // 列数
      const screenWidth = uni.getSystemInfoSync().windowWidth
      const screenHeight = uni.getSystemInfoSync().windowHeight
      this.cellWidth = screenWidth / colSize  // 格子宽度
      this.cellHeight = this.cellWidth  // 格子高度
	  // screenWidth * colSize / screenHeight
      for (let i = 0; i < rowSize; i++) {
        const row = []
        for (let j = 0; j < colSize; j++) {
          row.push({
            color: 'rgb(238,240,244)',
          })
        }
        this.grid.push(row)
      }
    },
    handleTouchStart(event) {
      const touch = event.touches[0]
      this.calculateCellIndex(touch)
    },
    handleTouchMove(event) {
      const touch = event.touches[0]
      this.calculateCellIndex(touch)
    },
    calculateCellIndex(touch) {
      const { clientX, clientY } = touch
      const rowIndex = Math.floor(clientY / this.cellHeight)
      const colIndex = Math.floor(clientX / this.cellWidth)

      this.grid[rowIndex][colIndex].color = 'rgb(130,161,250)'
      
      // Update the navigation bar color
      this.updateNavigationBarColor(this.grid[rowIndex][colIndex].color)
    },
    updateNavigationBarColor(color) {
      uni.setNavigationBarColor({
        frontColor: '#ffffff', // Text color of the navigation bar, set to white
        backgroundColor: color, // Background color of the navigation bar, set to the selected cell color
      })
    }
  }
}
</script>

<style>
.container {
  width: 100%;
  height: 100vh;
  overflow: hidden;
  position: relative;
}

.row {
  display: flex;
}

.cell {
  transition: background-color 0.1s linear;
}
</style>
