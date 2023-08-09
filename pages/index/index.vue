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
	<view v-if="initShow" class="overlay">
		<view class="overlay-container">
			<view class="overlay-chuMo">
				<view class="overlay-chuMo-text">
					屏幕触摸是否灵敏
				</view>
				<view class="overlay-chuMo-content">
					如折叠屏、曲面屏等机型，小程序界面无法完全覆盖全屏，清人工进行全屏的显示判断后再进行结果选择。</view>
				<button class="overlay-chuMo-btn" @click="confirmCheck($event)" >我已确认，开始检测</button>
			</view>
		</view>
	</view>
  </view>	
</template>

<script>
export default {
  name: "lxScreenCheck",
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
	  initShow:true ,
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
	  
      this.grid[rowIndex][colIndex].color = '#0AA2E9'
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
        frontColor: '#FFFFFF', // Text color of the navigation bar, set to white
        backgroundColor: color, // Background color of the navigation bar, set to the selected cell color
      })
    },
	// 点击"我已确认"按钮的事件处理函数
    confirmCheck(e) {
	  e.preventDefault();
	  this.initShow = false;
    }
  }
}
</script>

<style  lang="scss" scoped>
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
  background-color: #0AA2E9; /* Add a semi-transparent background to make it more visible */
  z-index: 5;
  &-container{
	  margin: 0 auto;
  }
  &-chuMo{ 
	  margin: 244rpx auto;
	  height: 320rpx;
	  width: 551rpx;
	  ext-align: center;
	  background-color: aqua;
	  align-items: center;
	  background-color: white;
	  border-radius: 10px;
	  padding-top: 40rpx;
	  background-color: rgba(255, 255, 255, 0.5); 
	  &-text{ 
		  margin-top: 12.0rpx;
		  text-align: center;
		  color: #333333;
		  font-weight: Bold;
	  }
	  &-btn { 
		  margin-top: 27rpx;
		  border-radius: 30rpx;
		  text-align: center; 
		  align-items: center; 
		  width: 500rpx ; 
		  height: 60rpx;
		  font-size: 14px;
		  background-color: #0AA2E9;
		  color: #FFFFFF;
		  opacity: 1;
		  mix-blend-mode: normal;  /* 修改为normal */
	   }
	   &-content{
		   margin-top: 38rpx;
		   font-size: 12px;
		   width: 504rpx;
		   height: 107.0rpx;
		   margin-left: 25rpx; 
		   line-height: 35rpx;
		   text-align: left;
		   font-weight: Regular;
	   }
  }
}

.overlay text {
  color: white;
  font-size: 24px;
  font-weight: bold;
}
</style>
