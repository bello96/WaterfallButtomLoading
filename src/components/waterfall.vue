<template>
  <ul class="waterfall" ref="waterfallbox">
    <li
      class="waterfall-item"
      ref="waterfallItem"
      :style="{ width: itemWidth + 'px' }"
      :key="index"
      v-for="(item, index) in newWaterInfo.length === 0
        ? waterInfo
        : newWaterInfo"
    >
      <slot :flow="item"></slot>
    </li>
    <li v-if="isShowNotYet || isPeriod" :style="baseLineStyle">
      -{{ noMoreText }}-
    </li>
  </ul>
</template>

<script>
export default {
  data() {
    return {
      newWaterInfo: [],
      fullindex: 0, // 添加数据标志
      isShowNotYet: false,
      // 设置暂无数据显示位置
      baseLineStyle: {
        position: "absolute",
        top: "0px",
        width: "100%",
        textAlign: "center",
        color: this.notMoreTextColor,
        paddingBottom: "5px",
      },
      itemWidth: 0, // 每个元素宽度
    };
  },
  props: {
    // 所有数据
    waterInfoAll: {
      type: Array,
      default: () => [],
    },
    // 类似分页的数据（累加）
    waterInfo: {
      type: Array,
      default: () => [],
    },
    isPeriod: {
      type: Boolean,
      default: false,
    },
    // 列数
    columns: {
      type: Number,
      default: 2,
    },
    // 间距
    offset: {
      type: Number,
      default: 5,
    },
    // 每次添加数量
    loadsSize: {
      type: Number,
      default: 20,
    },
    noMoreText: {
      type: String,
      default: "暂无更多",
    },
    notMoreTextColor: {
      type: String,
      default: "#aaa",
    },
  },
  watch: {
    // 监听全部数据 赋值
    waterInfoAll: {
      immediate: true,
      handler(newarr) {
        this.newWaterInfo = newarr.slice(0, this.loadsSize);
      },
    },
  },
  mounted() {
    this.waterfallbox = this.$refs.waterfallbox; // 容器
    this.scrollId = this.throttle(this.scrollCallback, 500, this.waterfallbox);
    this.waterfallbox.addEventListener("scroll", this.scrollId);
    // 每个元素的宽度
    this.itemWidth =
      (this.waterfallbox.offsetWidth - (this.columns - 1) * this.offset) /
      this.columns;
  },
  updated() {
    // 所有子元素
    this.childsNodes = this.$refs.waterfallItem;
    this.$nextTick(() => {
      this.getWaterfall(
        this.columns,
        this.offset,
        this.childsNodes,
        this.itemWidth
      );
    });
  },
  beforeDestroy() {
    this.removeScrollCallback(); // 移除滚动监听
  },
  methods: {
    // 布局
    getWaterfall(columns, offset, childsNodes, itemWidth) {
      let array = [];
      for (let i = 0; i < childsNodes.length; i++) {
        if (i < columns) {
          // 处理第一行
          childsNodes[i].style.top = 0 + "px";
          childsNodes[i].style.left =
            i % columns == 0
              ? itemWidth * i + "px"
              : itemWidth * i + offset * i + "px";
          array.push(childsNodes[i].offsetHeight); // 获得第一行子元素的高度集合
        } else {
          let minHeight = array[0]; // 假设第一项的高度为最小
          let index = 0; // 记录高度最小的下标
          for (let j = 0; j < array.length; j++) {
            //遍历数组array每一项，获得array中高度最小一项的下表值
            if (minHeight > array[j]) {
              minHeight = array[j];
              index = j;
            }
          }
          //设置当前子元素项的位置
          childsNodes[i].style.top = array[index] + offset + "px";
          childsNodes[i].style.left = childsNodes[index].offsetLeft + "px";
          //重新定义数组最小项的高度
          array[index] = array[index] + childsNodes[i].offsetHeight + offset;
        }
      }
    },
    // 添加数据
    addwaterInfo(index) {
      let arr = this.waterInfoAll.slice(
        this.loadsSize * index,
        this.loadsSize * index + this.loadsSize
      );
      this.newWaterInfo = this.newWaterInfo.concat(arr);
    },
    // 滚动回调
    scrollCallback(waterfallbox) {
      // 如果滚动区域高度与滚动偏移量之和 等于 整个滚动区域高度相等，说明到底了
      if (
        waterfallbox.clientHeight + parseInt(waterfallbox.scrollTop) + 10 >=
        waterfallbox.scrollHeight
      ) {
        if (this.newWaterInfo.length === 0 && !this.isPeriod) {
          this.$emit("bottomEvent");
          if (this.isPeriod) {
            this.setNotYetTop();
            this.removeScrollCallback();
          }
          return;
        }
        // 判断新的数据容器长度是否满了（与waterInfoAll相等）
        if (this.newWaterInfo.length >= this.waterInfoAll.length) {
          this.removeScrollCallback();
          this.setNotYetTop();
        } else {
          // 新容器还没有满 继续添加数据
          this.addwaterInfo(++this.fullindex);
        }
      }
    },
    // 设置暂无数据top值
    setNotYetTop() {
      let top = this.$refs.waterfallbox.scrollHeight;
      this.baseLineStyle.top = top + 16 + "px";
      this.isShowNotYet = true;
    },
    // 移除滚动监听
    removeScrollCallback() {
      let waterfallbox = this.$refs.waterfallbox;
      waterfallbox.removeEventListener("scroll", this.scrollId);
    },
    // 节流
    throttle(func, wait = 800, vdom) {
      let previous = 0,
        timer = null;
      return function anonymous() {
        let now = +new Date(),
          remaining = wait - (now - previous);
        if (remaining <= 0) {
          clearTimeout(timer);
          timer = null;
          previous = now;
          func.call(this, vdom);
        } else if (!timer) {
          timer = setTimeout(() => {
            clearTimeout(timer);
            timer = null;
            previous = +new Date();
            func.call(this, vdom);
          }, remaining);
        }
      };
    },
  },
};
</script>

<style scoped>
.waterfall {
  width: 100%;
  height: 100%;
  overflow: hidden;
  position: relative;
  background-color: #fff;
  list-style: none;
  padding: 0;
  margin: 0;
}
.waterfall:hover {
  /* 滚动条不占位置 */
  overflow-y: overlay;
}
.waterfall::-webkit-scrollbar {
  /* 滚动条整体样式  宽高分别对应横竖滚动条的尺寸大小*/
  width: 4px;
  height: 4px;
}
.waterfall::-webkit-scrollbar-track-piece {
  /* 滚动条没有滑块的轨道部分 */
  background: transparent;
}
.waterfall::-webkit-scrollbar-thumb {
  /*滚动条里面的滑块*/
  border-radius: 5px;
  background: #dddee0;
}
.waterfall-item {
  height: auto;
  font-size: 14px;
  border-radius: 3px;
  position: absolute;
  box-sizing: border-box;
  background-color: #f2f8fe;
}
</style>