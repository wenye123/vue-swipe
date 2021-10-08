<template>
  <div id="app">
    <Swipe
      class="my-swipe"
      :initIndex="currIndex"
      :showIndicator="true"
      ref="mySwipe"
      @change="change"
    >
      <SwipeItem v-for="(pic, index) in pics" :key="index">
        <img :src="pic" alt="" />
      </SwipeItem>
    </Swipe>

    <div class="tool">
      <span>{{ currIndex }}</span>
      <button @click="prev">上一页</button>
      <button @click="next">下一页</button>
      <button @click="redirectTo(3)">跳转到第四页</button>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { Swipe, SwipeItem } from "@/components/index";
// const { Swipe, SwipeItem } = require("@wenye123/vue-swipe");
// import "@wenye123/vue-swipe/dist/swipe.css";

@Component({
  components: { Swipe, SwipeItem },
})
export default class App extends Vue {
  $refs!: {
    mySwipe: any;
  };

  /** 当前索引 */
  currIndex = 1;
  /** 切换图片 */
  pics = [
    require("@/assets/1.jpg"),
    require("@/assets/2.jpg"),
    require("@/assets/3.jpg"),
    require("@/assets/4.jpg"),
  ];

  change(index: number) {
    this.currIndex = index;
  }
  prev() {
    this.$refs.mySwipe.prev();
  }
  next() {
    this.$refs.mySwipe.next();
  }
  redirectTo(index: number) {
    this.$refs.mySwipe.redirectTo(index);
  }
}
</script>

<style>
.my-swipe {
  width: 300px;
  height: 138px;
  margin: 30px auto 0 auto;
  background-color: #ccc;
}
.tool {
  margin-top: 30px;
  text-align: center;
}
.tool span,
.tool button {
  margin-right: 5px;
}
</style>
