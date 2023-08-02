<template>
  <view class="container" @touchstart="handleTouchStart" @touchmove="handleTouchMove">
    <view v-for="(row, rowIndex) in grid" :key="rowIndex" class="row">
      <view v-for="(col, colIndex) in row" :key="colIndex" class="cell" :style="{
        backgroundColor: col.color,
        width: cellWidth + 'px',
        height: cellHeight + 'px'
      }"></view>
    </view>
	<view v-if="xuanFu" class="overlay">
		<view style="height: 120px; width: 240px; text-align: center; background-color: aqua;  align-items: center;">请选择是否有异常</view>
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
      cellHeight: 0  ,// 格子高度
	  count : 0, 
	  flag :[] ,
	  xuanFu : false ,
	  row :0 , 
	  col : 0 ,
    }
  },
  mounted() {
    this.initGrid()
  },
  methods: {
    // 初始化格子图
    initGrid() {
      // const rowSize = 22 // 行数
      const colSize = 12 // 列数
      const screenWidth = uni.getSystemInfoSync().windowWidth
      const screenHeight = uni.getSystemInfoSync().windowHeight
      this.cellWidth = screenWidth / colSize  // 格子宽度
      this.cellHeight = this.cellWidth  // 格子高度\
	  // console.log(parseInt(screenWidth) /parseInt(colSize));
	  // console.log( parseInt(parseInt(screenHeight) / parseInt(this.cellHeight)) ) ;
	  // const colSize =  parseInt(screenHeight / this.cellHeight);
	  const rowSize = parseInt(screenHeight / this.cellHeight) + 1 ;
	  this.row = rowSize
	  this.col = colSize
      for (let i = 0; i < rowSize; i++) {
        const row = []
		const rowFlag = [];
        for (let j = 0; j < colSize; j++) {
		  row.push({
            color: 'rgb(238,240,244)',
          })
		  rowFlag.push(0);
        }
		this.flag.push(rowFlag);
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
	  if(this.flag[rowIndex][colIndex] == false) {
	    this.count ++ ;
	    this.flag[rowIndex][colIndex] = true 
	  }else { 
	  	
	  }
	  const rowSize = 22 // 行数
	  const screenWidth = uni.getSystemInfoSync().windowWidth
	  const screenHeight = uni.getSystemInfoSync().windowHeight
	  this.cellHeight = this.cellWidth  // 格子高度
	  const colSize =  screenHeight / this.cellHeight 
	  // console.log(this.count);
	  // if(this.count == 1) {
	  if(this.count == this.row * this.col) { 
		  // console.log("满了");
		  this.xuanFu = true 
	  }
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
.overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: rgba(0, 0, 0, 0.5); /* Add a semi-transparent background to make it more visible */
  z-index: 5;
}

.overlay text {
  color: white;
  font-size: 24px;
  font-weight: bold;
}
</style>
