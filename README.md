一个基于 vue2 的 swipe 库

#### demo

![demo](https://cdn.wenye123.com/blog/20211008114535.png)

#### script加载例子

```html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <meta
      name="viewport"
      content="initial-scale=1, maximum-scale=1, user-scalable=no, viewport-fit=cover"
    />
    <title>测试</title>
    <style>
      .my-swipe {
        width: 300px;
        height: 138px;
        margin: 30px auto 0 auto;
        background-color: #ccc;
      }
    </style>
  </head>
  <body>
    <div id="app">
      <swipe class="my-swipe">
        <swipe-item v-for="(pic, index) in pics" :key="index">
          <img :src="pic" alt="" />
        </swipe-item>
      </swipe>
    </div>
  </body>
  <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@wenye123/vue-swipe@latest/dist/swipe.js"></script>
  <link
    rel="stylesheet"
    href="https://cdn.jsdelivr.net/npm/@wenye123/vue-swipe@latest/dist/swipe.css"
  />
  <script>
    const { Swipe, SwipeItem } = window["vue-swipe"];
    new Vue({
      el: "#app",
      components: { swipe: Swipe, "swipe-item": SwipeItem },
      data() {
        return {
          pics: [
            "../2020-10/img/1.jpg",
            "../2020-10/img/2.jpg",
            "../2020-10/img/3.jpg",
            "../2020-10/img/4.jpg",
          ],
        };
      },
    });
  </script>
</html>
```

#### npm安装

install 

```bash
npm i -S @wenye123/vue-swipe
```

使用例子

```html
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
const { Swipe, SwipeItem } = require("@wenye123/vue-swipe");
import "@wenye123/vue-swipe/dist/swipe.css";

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
```

### Props

| 属性                 | 说明                                                         | 默认      |
| -------------------- | ------------------------------------------------------------ | --------- |
| initIndex            | 默认展示索引值                                               | 0         |
| loop                 | 是否循环模式                                                 | *true*    |
| autoPlay             | 是否自动播放                                                 | *false*   |
| interval             | 自动播放时候的切换间隔                                       | 2000      |
| slideTime            | 切换时长                                                     | 500       |
| threshold            | 滑动距离超过该阈值才会触发切换，值为0-1表示百分比，整数表示像素 | 0.3       |
| showIndicator        | 是否显示指示器                                               | *false*   |
| indicatorClick       | 指示器是否可点击切换                                         | *true*    |
| indicatorColor       | 指示器默认颜色                                               | \#fff     |
| indicatorActiveColor | 指示器激活后颜色                                             | cadetblue |

#### 事件

| 名称   | 描述             | 参数                |
| ------ | ---------------- | ------------------- |
| change | 切换item时候触发 | index: item的索引值 |

#### 方法

| 方法                      | 描述                               |
| ------------------------- | ---------------------------------- |
| prev()                    | 上一页                             |
| next()                    | 下一页                             |
| redirectTo(index: number) | 跳转到指定item，参数为item的索引值 |

