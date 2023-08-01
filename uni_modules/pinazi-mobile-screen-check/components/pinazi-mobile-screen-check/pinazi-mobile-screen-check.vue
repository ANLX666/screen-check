<template>
    <view class="pnz_fix_wrap">
        <view style="height: 100%">
            <view class="" style="height: 100%">
                <view class="content_wrap">
                    <view
                        class="content_item"
                        @tap="tapItem(index)"
                        :class="{ touched: item.touched, is_item: item.isItem }"
                        @touchmove="moveItem"
                        v-for="(item, index) in allData"
                        :key="index"
                    >
                        <text
                            v-if="item.isItem"
                            class="arrow_icon_base "
                            :class="[arrowIcon(item.direction)]"
                        >	
                        </text>
                    </view>
                </view>
                <view class="slot_absolute">

                    <slot>
                        <view style="font-size: 10px;color: #c9860c ;margin:10px"
                        >从此处开始</view
                        >
                        <view style="font-size: 10px; ;margin:10px"
                        >剩余格子:
                            <text >{{
                                    remainedCount()
                                }}</text></view
                        >
                        <view>
                            <text
                                style="font-size: 10px;margin:10px;color:#888"
                            >
                                触摸并滑动格子，点亮全部格子，则说明屏幕正常
                            </text>
                        </view>
                    </slot>
                </view>
            </view>
        </view>
    </view>
</template>
<script>
export default {
    name: "pinazi-mobile-screen-check",
    data() {
        return {
            allData: [],
            size: {
                x: 12,
                y: 21,
            },
            isFinished: false,
            itemSize: {
                width: 0,
                height: 0,
            },
            direction: ["top", "right", "bottom", "left"],
            currentDirection: 0,
            currentItemIndex: 0,
            step: 1,
            currentMaxY: 0,
            currentMaxX: 0,
            openGid: "",
        };
    },
    mounted() {
        this.init();
    },
    methods: {
        arrowIcon(direction) {
            switch (direction) {
                case 0:
                    return "arrow_top";
                case 1:
                    return "arrow_right";
                case 2:
                    return "arrow_bottom";
                case 3:
                    return "arrow_left";
            }
        },
        init() {
            console.log(uni.getSystemInfoSync())
            //获得数据
            for (var j = 0; j <= this.size.y; j++) {
                for (var i = 0; i <= this.size.x; i++) {
                    this.allData.push({
                        x: i,
                        y: j,
                        direction: 0,
                        isItem: false,
                        touched: false,
                    });
                }
            }
            //算出每个item的宽高
            var column_height;
            // #ifdef H5
            column_height = uni.getSystemInfoSync().screenHeight / (this.size.y + 1)
            // #endif
            // #ifdef MP-WEIXIN || APP-PLUS
            column_height = uni.getSystemInfoSync().windowHeight / (this.size.y + 1)
            // #endif
			console.log(column_height);
            this.itemSize = {
                width: uni.getSystemInfoSync().screenWidth / (this.size.x + 1),
                height: column_height
            };
            //将需要touch到的元素集合算出来
            this.addTouchItem();
        },
        addTouchItem() {
            // 先初始化当前的item
            this.currentItemIndex = this.allData.findIndex((value, index) => {
                return value.x === this.size.x / 2 && value.y === this.size.x;
            });
            this.addOneLineItems();
        },
        addOneLineItems(log) {
            // console.log(this.allData[this.currentItemIndex]);
            if (this.allData[this.currentItemIndex] === undefined) {
                // 手动将留下的两个挂上item
                for (var i = 1; i < 1; i++) {
                    var currentItemIndex = this.allData.findIndex((value) => {
                        return value.x === this.size.x && value.y === i;
                    });
					console.log(this.currentDirection)
                    this.allData[currentItemIndex].isItem = true;
                    this.allData[currentItemIndex].direction =
                        this.currentDirection;
                }
                //此时应跳出循环
                return false;
            }
            this.allData[this.currentItemIndex].isItem = true;
            if (log === 1) {
                console.log(this.currentMaxY);
                console.log(this.allData[this.currentItemIndex]);
            }
            this.allData[this.currentItemIndex].direction =
                this.currentDirection;
            var step = this.step;
            if (
                this.currentMaxY === 0 ||
                (this.currentMaxY === 6 && this.currentDirection === 2)
            ) {
                step = step * 2;
            }
            if (log === 1) {
                console.log("step: " + step);
            }
            var currentItem = this.allData[this.currentItemIndex];
            // 当前的currentMax加上
            var currentLength = 0;
            if (log) {
                console.log("this.currentMaxY: " + this.currentMaxY);
            }
            if (this.currentDirection === 0 || this.currentDirection === 2) {
                currentLength = step + this.currentMaxY;
                this.currentMaxY = currentLength;
            } else {
                currentLength = step + this.currentMaxX;
                this.currentMaxX = currentLength;
            }
            if (log) {
                console.log("currentLength: " + currentLength);
            }
            for (var i = 0; i < currentLength; i++) {
                var nowItemIndex = this.allData.findIndex((value) => {
                    return this.getCurrentByDrirection(
                        value,
                        currentItem,
                        i,
                        this.currentDirection
                    );
                });
                if (log) {
                    console.log("currentLength: " + currentLength);
                    console.log(nowItemIndex);
                    console.log(this.currentDirection);
                    console.log(this.allData[nowItemIndex]);
                }
                if (nowItemIndex >= 0) {
                    this.allData[nowItemIndex].isItem = true;
                    this.allData[nowItemIndex].direction =
                        this.currentDirection;
                }
            }
            //把当前的currentIndex索引移到上一个上
            this.currentItemIndex = this.allData.findIndex((value) => {
                return this.getCurrentByDrirection(
                    value,
                    currentItem,
                    currentLength,
                    this.currentDirection
                );
            });
            this.currentDirection = (this.currentDirection + 1) % 4;
            return this.addOneLineItems();
        },
        getCurrentByDrirection(value, currentItem, i, direction) {
            switch (direction) {
                case 0:
                    return (
                        value.x === currentItem.x && value.y === currentItem.y - i
                    );
                case 1:
                    return (
                        value.x === currentItem.x + i && value.y === currentItem.y
                    );
                case 2:
                    return (
                        value.x === currentItem.x && value.y === currentItem.y + i
                    );
                case 3:
                    return (
                        value.x === currentItem.x - i && value.y === currentItem.y
                    );
            }
        },
        remainedCount() {
            //判断是否还有_isItem 但还没有touch
            var notFinished = this.allData.filter((value, index) => {
                return value.isItem && !value.touched;
            });
            return notFinished.length;
        },
        markFinished(selectedItemIndex) {
            if (selectedItemIndex >= 0) {
                if (this.allData[selectedItemIndex].isItem) {
                    this.allData[selectedItemIndex].touched = true;
                }
                //判断是否还有_isItem 但还没有touch
                if (this.remainedCount() === 0 && !this.isFinished) {
                    this.isFinished = true;
                    //说明已经结束
                    this.emitSucceed()
                }
            }
        },
        moveItem(e) {
            //算出浮到的元素的x,y
            var nowX = e.changedTouches[0].pageX;
            var nowY = e.changedTouches[0].pageY + uni.getSystemInfoSync().windowTop;
			console.log(nowY);
            var xIndex = Math.floor(nowX / this.itemSize.width);
            var yIndex = Math.floor(nowY / this.itemSize.height);
            console.log(xIndex,yIndex);
            var selectedItemIndex = this.allData.findIndex((value) => {
                return value.x === xIndex && value.y === yIndex;
            });
            //取得浮到的元素的index
            this.markFinished(selectedItemIndex);
        },
        tapItem(index) {
            this.markFinished(index);
        },
        jump() {
            uni.showModal({
                content: "确定触摸屏异常吗？",
                showCancel: true,
                success: () => {
                    this.emitFailed()
                },
            });
        },
        emitFailed() {
            // alert("failed")
            this.$emit("failed")
        },
        emitSucceed() {
            // alert("succeed")
            this.$emit("succeed")
        },
    },
};
</script>
<style lang="scss" scoped>
.pnz_fix_wrap{
    position: fixed;
    left:0;
    right: 0;
    bottom: 0;
    top:0;
    background: white;
    z-index: 1000;
    overflow: hidden;
}
.content_wrap {
    height: 100%;
    font-size: 12px;
    display: flex;
    flex-wrap: wrap;
    align-content: stretch;
}
.content_item {
    display: flex;
    justify-content: center;
    min-width: 7.69%;
    box-sizing: border-box;
    &.is_item {
        box-shadow: 0 0 1px 1px #dedede;
    }
    &.touched {
        background-color: #bbb;
        color: white;
    }
}
.slot_absolute {
    position: absolute;
    left: calc(50% - 50px);
    width: 100px;
    line-height: 14px;
    text-align: center;
    top: calc(50% + 60px);
}
.arrow_icon_base{
    background-position:center;
    background-repeat: no-repeat;
    width:100%;
    display: flex;
    align-items: center;
    justify-content: center;
    background-size: 24px 24px;
}
.arrow_top{
    background-image: url("data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAgCAYAAABzenr0AAAAAXNSR0IArs4c6QAAANZJREFUWEftlEEKAjEMRZPr6GJmo/dxoInoeRSTgt5HN85CrxMpuOhCsa3ibNJdIf3/8VqKMPHCifvBAdyAG3ADXxkIIQzpI4sxHls/tGYAZp6Z2S0VI+JcRO4tEM0ARHQFgO5ZOqpq/zeAEMIOETd5oZntY4zbWohqA8y8MrOXd46Ig4icaiCqAJi5M7Ok/u1CxF5ExlKIKgAiOgPA4kP4RVWXPwcgogMAcGGwqOq6ZLbKQB5IRJbvVbUpq+lQKnaAyQ2UPLCSmeY3UBJeMuMAbsANuIEH1IFDId8utHMAAAAASUVORK5CYII=");
}
.arrow_right{
    background-image: url("data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAgCAYAAABzenr0AAAAAXNSR0IArs4c6QAAAMhJREFUWEftlEEKwjAQRTO3FDeCkiD0PsXJDwpeR0HqQo/zRakgXUiaVOpish7mPV5CxM18ZGa+MwErYAWsQHEB730EEGo/smKBEAKdcxcRCap6LhWpFXhxSTYA2hKJSQR6iSOA1ViJbIE++df9JO8kFymla67IpAIfUB9jTDkSPxEguQGwn0OgE5Glqt5y4M+Z7ALDhcM3QbIF0OSC33OTCIjIWlUPY+FTFDiJiFfVrgReK7CLMW5LwdVXUAs2AStgBazA3xR4AMFrUCH+4ZvCAAAAAElFTkSuQmCC");
}
.arrow_left{
    background-image: url("data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAgCAYAAABzenr0AAAAAXNSR0IArs4c6QAAANZJREFUWEftlEEKwjAQRecfRxcWQa8jKJ2M9D4FmaDgRrzlSBdCkULStLabyTYk/+X9IaCVF1bOJwdwA27ADcxigJk1xhhKPrVJACJyMDMlor2qFt1VdKh7KTM3ANrvqxcFYOYngFNf+SIAdV3vALwBbFJ95wJlVxBCqIkopoLHVpINwMwXAPfVALpgEdma2YuIqhTI7BX0A5m5BdAsPoT9QBE5m9ljbOe/5rJnYEi5iFRm1g3mMVf5rADfy0IIN1W9puZiaH+SgZLAvxiYAuIG3IAbcAMfCBs8IcVSESQAAAAASUVORK5CYII=");
}
.arrow_bottom{
    background-image: url("data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAgCAYAAABzenr0AAAAAXNSR0IArs4c6QAAAOVJREFUWEftlU0KwjAQhfN6G10oil5HN50U9DwKnelGr6MoutDbdCSgUES0iX+bySZZJO97fLQJ3J8H/sx3VsAMmAEzkGzAe6/NS4yZk7KSDgWwFfi5ASJiANTm8VJVERHfZm/UN+C93znnBi+C98w8bAMPe6IKFEUxUtXNs3AA47Ist18pEEKJaAZg8QigqnMRWbaFRxu4BRPRCsCkCVLVtYhMY+DJBa4mTgA6Ya2qZxHpxsLfKpDneS/LskMIqeu6X1XV8acFrrdhHmZmrlLgbxlIBd6fi/oNPwVt5lgBM2AGzMAFs7pJISdlO+wAAAAASUVORK5CYII=");
}
@media screen and (min-width: 340px) {
    .absolute {
        top: calc(50% + 70px);
    }
}
@media screen and (min-width: 600px) {
    .absolute {
        top: calc(50% + 100px);
    }
}
@media screen and (min-width: 700px) {
    .absolute {
        top: calc(50% + 120px);
    }
}
</style>
