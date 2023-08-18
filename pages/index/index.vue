<template>
    <view class="container" @touchstart="handleTouchStart" @touchmove="handleTouchMove">
        <view v-for="(row, rowIndex) in grid" :key="rowIndex" class="row">
            <view v-for="(col, colIndex) in row" :key="colIndex" class="cell" :style="{
        backgroundColor: col.color,
        width: cellWidth + 'px',
        height: cellHeight + 'px'
      }"></view>
        </view>
        <view v-if="!initShow" class="overlay2">
            <view class="goback" @click.stop="goback"> 
                <u-icon name="arrow-left"></u-icon> 
                <text>返回</text>
            </view>
            <view class="overlay2-container">
                <view class="overlay2-chuMo">
                    <view class="overlay2-chuMo-text">
                        手指划过所有灰色区域使之变为蓝色
                    </view>
                    <view class="overlay2-chuMo-content">
                        如果有异常请点击返回
                    </view>
                </view>
            </view>
        </view>
        <view v-if="initShow" class="overlay">
            <view class="overlay-container">
                <view class="overlay-chuMo">
                    <view class="overlay-chuMo-text">
                        屏幕触摸是否灵敏
                    </view>
                    <view class="overlay-chuMo-content">
                        如折叠屏、曲面屏等机型，小程序界面无法完全覆盖全屏，请人工进行全屏的显示判断后再进行结果选择。</view>
                    <button class="overlay-chuMo-btn" @click="confirmCheck($event)">我已确认，开始检测</button>
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
                grid: [], // 格子图
                cellWidth: 0, // 格子宽度
                cellHeight: 0, // 格子高度
                count: 0,
                flag: [],
                xuanFu: false,
                row: 0,
                col: 0,
                initShow: true,
				screenWidth:0,
				screenHeight:0,
            }
        },
        mounted() {
			this.getSystemInfo()
			this.initGrid();
        },
        methods: {
			getSystemInfo() {
		        const systemInfo = uni.getSystemInfoSync()
		        this.screenWidth = systemInfo.windowWidth
		        this.screenHeight = systemInfo.windowHeight
			},
            // 初始化格子图
            initGrid() {
				const colSize = 12 // 列数
                // const rowSize = 22 // 行数
                this.cellWidth = this.screenWidth / colSize // 格子宽度
                this.cellHeight = this.cellWidth // 格子高度\
                // console.log(parseInt(screenWidth) /parseInt(colSize));
                // console.log( parseInt(parseInt(screenHeight) / parseInt(this.cellHeight)) ) ;
                // const colSize =  parseInt(screenHeight / this.cellHeight);
                const rowSize = parseInt(this.screenHeight / this.cellHeight) + 1;
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
                const {
                    clientX,
                    clientY
                } = touch
                const rowIndex = Math.floor(clientY / this.cellHeight)
                const colIndex = Math.floor(clientX / this.cellWidth)

                this.grid[rowIndex][colIndex].color = '#0aa2e9'
                // Update the navigation bar color
                this.updateNavigationBarColor(this.grid[rowIndex][colIndex].color)
                if (this.flag[rowIndex][colIndex] == false) {
                    this.count++;
                    this.flag[rowIndex][colIndex] = true
                } else {

                }
                const rowSize = 22 // 行数
                this.cellHeight = this.cellWidth // 格子高度
                const colSize = this.screenHeight / this.cellHeight
                // console.log(this.count);
                // if(this.count == 1) {
                if (this.count == this.row * this.col) {
                    console.log("满了");
                    this.$emit('setInfo', 1)
                    // this.$router.push('/pages/index/index')
                }
            },
            updateNavigationBarColor(color) {
                uni.setNavigationBarColor({
                    frontColor: '#ffffff', // Text color of the navigation bar, set to white
                    backgroundColor: color, // Background color of the navigation bar, set to the selected cell color
                })
            },
            // 点击"我已确认"按钮的事件处理函数
            confirmCheck(e) {
                e.preventDefault();
                this.initShow = false;
            },
            // 返回
            goback() {
                console.log('goback')
                this.$emit('setInfo', 0)
            }
        }
    }
</script>

<style lang="scss" scoped>
    .container {
        width: 100%;
        height: 100vh;
        overflow: hidden;
        position: relative;
    }
    
    .goback{
        display: flex;
        position: absolute;
        top: 100rpx;
        left: 50rpx;
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
        background-color: #0AA2E9;
        /* Add a semi-transparent background to make it more visible */
        z-index: 5;

        &-container {
            margin: 0 auto;
        }

        &-chuMo {
            margin: 244rpx auto;
            height: 350rpx;
            width: 587rpx;
            ext-align: center;
            background-color: aqua;
            align-items: center;
            background-color: white;
            border-radius: 10px;
            padding-top: 43rpx;
            background-color: rgba(255, 255, 255, 0.7);

            &-text {
                text-align: center;
                color: #333333;
                font-weight: 700;
                font-size: 15px;
                opacity: 1;
            }

            &-btn {
                margin-top: 27rpx;
                border-radius: 30rpx;
                width: 500rpx;
                height: 60rpx;
                font-size: 14px;
                background-color: #0AA2E9;
                color: #FFFFFF;
                opacity: 1;
                border: none;
                mix-blend-mode: normal;
                /* 修改为normal */
                display: flex;
                justify-content: center;
                align-items: center;
            }

            &-btn::after {
                border: none;
            }

            &-content {
                margin-top: 45rpx;
                font-size: 12px;
                width: 504rpx;
                height: 107.0rpx;
                margin-left: 34rpx;
                line-height: 40rpx;
                text-align: left;
                font-weight: 400;
            }
        }
    }

    .overlay2 {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        display: flex;
        background: rgba(255, 255, 255, 0);
        z-index: 5;

        &-container {
            margin: 0 auto;
        }

        &-chuMo {
            margin: 407rpx auto;
            height: 320rpx;
            width: 551rpx;
            ext-align: center;
            background-color: aqua;
            align-items: center;
            background-color: white;
            border-radius: 10px;
            padding-top: 40rpx;
            background-color: rgba(255, 255, 255, 0);

            &-text {
                margin-top: 12.0rpx;
                text-align: center;
                color: #333333;
                font-weight: Bold;
                font-size: 15px;
            }

            &-btn {
                border-radius: 15px;
                text-align: center;
                align-items: center;
                justify-content: center;
                font-size: 14px;
                background-color: #0AA2E9;
                color: #FFFFFF;
                opacity: 1;
                border: none;
                mix-blend-mode: normal;
                /* 修改为normal */
            }

            &-content {
                margin-top: 38rpx;
                font-size: 12px;
                width: 504rpx;
                height: 107.0rpx;
                margin-left: 25rpx;
                font-weight: Regular;
                line-height: 35rpx;
                text-align: center;
                font-weight: Regular;
            }
        }
    }

    .overlay text {
        color: white;
        font-size: 24px;
        font-weight: bold;
    }

    .overlay-xuanFu {
        width: 400px;
        height: 200px;
        background-color: #0AA2E9;
        z-index: 100;
    }

    .overlay-xuanFu text {
        color: #333333;
        font-size: 24px;
        font-weight: bold;
    }
</style>
