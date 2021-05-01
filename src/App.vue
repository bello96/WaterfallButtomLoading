<template>
  <div class="box">
    <waterfall-flow
      :columns="columns"
      :offset="offset"
      :noMoreText="noMoreText"
      :notMoreTextColor="notMoreTextColor"
      :loadsSize="loadsSize"
      :waterInfoAll="waterInfoAll"
    >
      <template v-slot="{ flow }">
        <!-- 使用者自定义渲染风格 -->
        <div class="itembox">
          <span class="wordcolor">{{ flow.content }}</span>
        </div>
      </template>
    </waterfall-flow>
  </div>
</template>

<script>
/* 
参数说明
columns => 列数（默认：2）
offset => 元素之间间距（默认：5）
noMoreText => 暂无更多提示信息（默认：暂无更多）
notMoreTextColor => 暂无更多文本颜色（默认：#aaa）
loadsSize => 每次添加数量（默认：20）

waterInfoAll => 全部数据

waterInfo => 分页式添加数据容器
bottomEvent => 触底回调函数（配合分页式）
isPeriod => 没有数据标志（没有则传true 配合分页式）
*/
import waterfallFlow from "./components/waterfall.vue";
export default {
  name: "App",
  data() {
    return {
      columns: 2, // 列数
      offset: 5, // 元素之间的间距
      noMoreText: "没有更多了", // 暂无更多提示信息
      notMoreTextColor: "#aaa", // 暂无更多文字颜色
      waterInfoAll: [], // 一次性全部数据
      loadsSize: 30, // 每次触底加载数量
    };
  },
  components: {
    waterfallFlow,
  },
  created() {
    // 模拟请求数据
    fetch("./data.json")
      .then((res) => res.json())
      .then((data) => {
        this.waterInfoAll = data;
      });
  },
};
</script>

<style lang="less" scoped>
.box {
  width: 400px;
  height: 600px;
  margin: 0 auto;
  border: 1px solid #aaa;
  border-radius: 5px;
  .itembox {
    text-indent: 2em;
    background-color: #f1f3f4;
    width: 100%;
    height: 100%;
    box-sizing: border-box;
    padding: 8px;
    border-radius: 5px;
    .wordcolor {
      color: #282c34;
      text-indent: 2em;
      font-size: 16px;
      list-style: none;
      margin: 5px 0;
      padding: 0;
    }
  }
}
</style>