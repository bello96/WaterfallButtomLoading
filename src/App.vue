<template>
  <div class="box">
    <waterfall-flow
      :waterInfo="waterInfo"
      :columns="columns"
      :offset="offset"
      :loadsNum="loadsNum"
      :noMoreText="noMoreText"
      :notMoreTextColor="notMoreTextColor"
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
import waterfallFlow from "./components/waterfall.vue";
export default {
  name: "App",
  data() {
    return {
      waterInfo: [], // 数据 （必传）
      columns: 4, // 列数
      offset: 5, // 元素之间的间距
      loadsNum: 200, // 每次触底加载数量
      noMoreText: "没有更多了", // 暂无更多提示信息
      notMoreTextColor: "#aaa", // 暂无更多文字颜色
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
        this.waterInfo = data;
      });
  },
};
</script>

<style lang="less" scoped>
.box {
  width: 600px;
  height: 800px;
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