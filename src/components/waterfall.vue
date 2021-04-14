<template>
  <ul class="waterfall" ref="waterfallbox">
    <li
      class="waterfall-item"
      ref="waterfallItem"
      :style="{ width: `${(1 / columns) * 100}%` }"
      :key="index"
      v-for="(item, index) in newWaterInfo"
    >
      {{ item.content }}
    </li>
    <li v-if="isshownotyet" :style="baselinestyle">-我是有底线的-</li>
  </ul>
</template>

<script>
export default {
  data() {
    return {
      newWaterInfo: [], // 新容器
      fullindex: 0, // 添加数据标志
      isshownotyet: false, // 是否显示我有底线
      baselinestyle: {
        position: "absolute",
        top: "0px",
        width: "100%",
        textAlign: "center",
        color: "#aaa",
      },
    };
  },
  props: {
    waterInfo: {
      type: Array,
      default: () => [],
    },
    columns: {
      type: Number,
      default: 2,
    },
    offset: {
      type: Number,
      default: 5,
    },
  },
  created() {
    // 第一次加载渲染截取前20个
    this.newWaterInfo = this.waterInfo.slice(0, 20);
  },

  mounted() {
    // 布局设置
    let childsNode = this.$refs.waterfallItem; //获取所有子元素dom
    this.getWaterfall(this.columns, this.offset, childsNode);
    // 给最外层盒子注册滚动事件
    /*
     * clientHeight 滚动可视区域高度（最外层盒子）
     * scrollTop 当前滚动位置 （滚动偏移量）
     * scrollHeight 整个滚动高度
     */
    let waterfallbox = this.$refs.waterfallbox;
    waterfallbox.addEventListener(
      "scroll",
      this.throttle(this.scrollCallback, 1000, waterfallbox)
    );
  },
  methods: {
    // 瀑布流布局结构处理函数
    getWaterfall(columns, offset, childsNodes) {
      /*
        参数说明
        columns  => 定义布局的列数
        offset => 元素与元素之间的偏移量
        childsNodes => 所有子元素节点
      */
      let array = []; //定义空数组存储元素高度
      for (let i = 0; i < childsNodes.length; i++) {
        //遍历整个子元素的DOM集合
        if (i < columns) {
          // 处理第一行
          childsNodes[i].style.top = 0 + "px";
          childsNodes[i].style.left =
            i % columns == 0
              ? childsNodes[0].offsetWidth * i + "px"
              : childsNodes[0].offsetWidth * i + offset + "px";
          array.push(childsNodes[i].offsetHeight); // 获得第一行子元素的高度集合
        } else {
          let minHeight = array[0]; // 假设第一项的高度为最小
          let index = 0; // 记录高度最小的下标
          for (let j = 0; j < array.length; j++) {
            //遍历数组array每一项，目的获得array的最小值和其索引值
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
      let arr = this.waterInfo.slice(20 * index, 20 * index + 20);
      this.newWaterInfo = this.newWaterInfo.concat(arr);
      // 重新渲染
      this.$nextTick(() => {
        let childsNode = this.$refs.waterfallItem; //获取所有子元素dom
        this.getWaterfall(this.columns, this.offset, childsNode);
      });
    },
    // 滚动回调
    scrollCallback(waterfallbox) {
      // 如果滚动区域高度与滚动偏移量之和 等于 整个滚动区域高度相等，说明到底了
      if (
        waterfallbox.clientHeight + parseInt(waterfallbox.scrollTop) + 2 >=
        waterfallbox.scrollHeight
      ) {
        // 判断新的数据容器长度是否满了（与waterInfo相等）
        if (
          this.newWaterInfo.length >= this.waterInfo.length &&
          !this.isscrool
        ) {
          // 已满 提示暂无更多
          let top = this.$refs.waterfallbox.scrollHeight;
          this.baselinestyle.top = top + 15 + "px";
          this.isshownotyet = true;
          this.isscrool = true; // 一但满了 就不再处理
        } else {
          // 新容器还没有满 继续添加数据
          this.addwaterInfo(++this.fullindex);
        }
      }
    },
    /* 函数节流 */
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

<style>
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
  width: 5px;
  height: 5px;
}
.waterfall::-webkit-scrollbar-track-piece {
  /* 滚动条没有滑块的轨道部分 */
  background: transparent;
}
.waterfall::-webkit-scrollbar-thumb {
  /*滚动条里面的滑块*/
  border-radius: 5px;
  background: rgb(193, 193, 193);
}
.waterfall-item {
  width: 30%;
  height: auto;
  font-size: 16px;
  border-radius: 3px;
  padding: 10px;
  position: absolute;
  box-sizing: border-box;
  text-indent: 2em;
  background-color: rgb(241, 243, 244);
}
</style>