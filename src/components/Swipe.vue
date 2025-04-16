<template>
  <div class="swipe" ref="swipe">
    <ul class="list" ref="list">
      <slot></slot>
    </ul>
    <ul class="dots" ref="dots" v-if="showIndicator">
      <li
        class="dot"
        v-for="(o, i) of items"
        :key="i"
        :style="{
          'background-color':
            currIndex === i ? indicatorActiveColor : indicatorColor,
        }"
        @click="handleIndicatorClick(i)"></li>
    </ul>
  </div>
</template>

<script lang="ts">
import { Component, Emit, Prop, Vue } from "vue-property-decorator";

@Component({
  components: {},
})
export default class Swipe extends Vue {
  /** 默认展示索引值 */
  @Prop({ type: Number, default: 0 }) readonly initIndex!: number;
  /** 是否循环模式 */
  @Prop({ type: Boolean, default: true }) readonly loop!: boolean;
  /** 是否自动播放 */
  @Prop({ type: Boolean, default: false }) readonly autoPlay!: boolean;
  /** 自动播放时候的切换间隔 */
  @Prop({ type: Number, default: 2000 }) readonly interval!: number;
  /** 切换时长 */
  @Prop({ type: Number, default: 500 }) readonly slideTime!: number;
  /** 滑动距离超过该阈值才会触发切换，值为0-1表示百分比，整数表示像素 */
  @Prop({ type: Number, default: 0.3 }) readonly threshold!: number;
  /** 是否显示指示器 */
  @Prop({ type: Boolean, default: false }) readonly showIndicator!: boolean;
  /** 指示器是否可点击切换 */
  @Prop({ type: Boolean, default: true }) readonly indicatorClick!: boolean;
  /** 指示器默认颜色 */
  @Prop({ type: String, default: "#fff" }) readonly indicatorColor!: string;
  /** 指示器激活后颜色 */
  @Prop({ type: String, default: "cadetblue" })
  readonly indicatorActiveColor!: string;

  $refs!: {
    swipe: HTMLDivElement;
    list: HTMLUListElement;
    dots: HTMLUListElement;
  };

  /** item长度 */
  itemW = 0;
  /** 定时器句柄 */
  timer = -1;
  /** 当前索引 */
  currIndex = 0;
  /** item dom列表 */
  items: HTMLLIElement[] = [];
  /** 列表移动距离 */
  translateX = 0;
  /** 是否手指操作 */
  isTouch = false;
  /** 手指操作移动距离 */
  touchMoveX = 0;

  mounted(): void {
    // 获取滚动列表
    this.items = [
      ...this.$refs.swipe.querySelectorAll(".list > .swipe-item"),
    ] as HTMLLIElement[];
    // 参数校验
    this.checkProp();
    // 设置大小
    this.$refs.list.style.width = `${this.items.length * 100}%`;
    this.items.forEach((item) => {
      item.style.width = `${100 / this.items.length}%`;
    });
    // 获取单次滑动长度
    this.itemW = this.$refs.swipe.offsetWidth;
    // 初始化默认展示页面
    this.currIndex = this.initIndex;
    this.resetListX();
    // 定时轮播
    this.autoPlayFn();
    // 手指操作
    this.touchHandle();
  }

  beforeDestroy() {
    clearInterval(this.timer);
  }

  /** 参数校验 **/
  private checkProp() {
    if (this.items.length === 0) throw new Error("SwipeItem子组件不能为空");
    if (!(this.initIndex >= 0 && this.initIndex <= this.items.length - 1))
      throw new Error("初始索引不正确");
    if (this.threshold < 0) throw Error("threshold必须为正数");
  }

  /** 上一页 **/
  @Emit("change")
  prev() {
    // 非循环模式到边界后不进行操作
    if (this.loop === false && this.currIndex === 0) return;
    // 非手指滑动下(操作放在touchmove) 重置位置 && 如果当前是第一页则将最后一页移动到最前面
    if (this.isTouch === false) {
      this.resetListX();
      if (this.currIndex === 0) this.setEdgeX("last");
    }
    // 列表向前移动一页，这里使用setimeout是为了避免和前面的重置操作同步执行 返回promise是为了搭配@Emit的用法
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        this.translateListX("prev");
        // 修改页码
        this.currIndex = this.getIndex("prev");
        resolve(this.currIndex);
      }, 10);
    });
  }

  /** 下一页 **/
  @Emit("change")
  next() {
    // 非循环模式到边界后不进行操作
    if (this.loop === false && this.currIndex === this.items.length - 1) return;
    // 非手指滑动下(操作放在touchmove) 重置位置 && 如果当前是最后一页则将第一页移动到最后
    if (this.isTouch === false) {
      this.resetListX();
      if (this.currIndex === this.items.length - 1) this.setEdgeX("first");
    }
    // 列表向后移动一页
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        this.translateListX("next");
        // 修改页码
        this.currIndex = this.getIndex("next");
        resolve(this.currIndex);
      }, 10);
    });
  }

  /** 当前页 **/
  curr() {
    this.translateListX("curr");
  }

  /** 跳转到指定页面 **/
  @Emit("change")
  redirectTo(toIndex: number) {
    let diff = toIndex - this.currIndex;
    if (diff > 0) {
      for (; diff > 0; diff--) {
        this.next();
      }
    } else {
      for (; diff < 0; diff++) {
        this.prev();
      }
    }
    return this.currIndex;
  }

  /** 设置边界位置 first|last **/
  private setEdgeX(edge: "first" | "last") {
    if (edge === "first") {
      // 将第一页移动到最后一页后面
      const firstItem = this.items[0];
      const translateX = this.items.length * this.itemW;
      firstItem.style.transition = "";
      firstItem.style.transform = `translateX(${translateX}px) translateZ(0px)`;
    } else if (edge === "last") {
      // 将最后一页移到第一页的前面
      const lastItem = this.items[this.items.length - 1];
      const translateX = -this.items.length * this.itemW;
      lastItem.style.transition = "";
      lastItem.style.transform = `translateX(${translateX}px) translateZ(0px)`;
    } else {
      throw new Error("无效的edge");
    }
  }

  /** 重置列表和子项位置 **/
  private resetListX() {
    // 去掉子项移动位置
    this.items[this.currIndex].style.transition = "";
    this.items[this.currIndex].style.transform = "";
    // 将列表定位在本来就应该的位置 如果是手指操作则加上手指滑动距离
    this.translateX = -this.currIndex * this.itemW;
    if (this.isTouch === true) this.translateX += this.touchMoveX;
    this.$refs.list.style.transition = "";
    this.$refs.list.style.transform = `translateX(${this.translateX}px) translateZ(0px)`;
  }

  /** 设置列表移动 prev next curr **/
  private translateListX(type: "prev" | "next" | "curr") {
    const width =
      type === "prev" ? this.itemW : type === "next" ? -this.itemW : 0;
    if (this.isTouch === true) this.translateX -= this.touchMoveX; // 手指滑动模式下去掉因为触摸滑动的距离
    this.translateX += width;
    this.$refs.list.style.transition = `all ${this.slideTime / 1000}s`;
    this.$refs.list.style.transform = `translateX(${this.translateX}px) translateZ(0px)`;
    // 手指模式下重置手指参数
    if (this.isTouch === true) {
      this.isTouch = false;
      this.touchMoveX = 0;
    }
  }

  /** 获取索引 **/
  private getIndex(type: "prev" | "next") {
    if (type === "prev") {
      return this.currIndex - 1 >= 0
        ? this.currIndex - 1
        : this.items.length - 1;
    } else if (type === "next") {
      return this.currIndex + 1 >= this.items.length ? 0 : this.currIndex + 1;
    } else {
      throw new Error("不支持的type");
    }
  }

  /** 自动播放 **/
  private autoPlayFn() {
    if (this.autoPlay === true) {
      this.timer = setInterval(() => {
        this.next();
      }, this.interval);
    }
  }

  /** 手指滑动 **/
  private touchHandle() {
    let startX = 0;
    // touchstart
    this.$refs.list.addEventListener("touchstart", (e) => {
      startX = e.targetTouches[0].pageX;
      // 重置位置
      this.resetListX();
      // 手指触摸时候停止定时器
      if (this.timer !== -1) clearInterval(this.timer);
    });
    // touchmove
    this.$refs.list.addEventListener("touchmove", (e) => {
      // 标记为touch模式
      this.isTouch = true;
      // 获取移动距离
      this.touchMoveX = e.targetTouches[0].pageX - startX;
      // 阻止浏览器默认行为防止滑动影响到其他区域
      if (this.touchMoveX > 20) {
        e.preventDefault();
      }
      // 判断边界
      if (this.touchMoveX > 0 && this.currIndex === 0) {
        if (this.loop === true) {
          this.setEdgeX("last");
        } else {
          this.touchMoveX = 0;
          this.isTouch = false;
          return;
        }
      } else if (
        this.touchMoveX < 0 &&
        this.currIndex === this.items.length - 1
      ) {
        if (this.loop === true) {
          this.setEdgeX("first");
        } else {
          this.touchMoveX = 0;
          this.isTouch = false;
          return;
        }
      }
      // 按照手指移动的距离同步移动
      this.resetListX();
    });
    // touchend
    this.$refs.list.addEventListener("touchend", (e) => {
      // 手指移动过了才执行
      if (this.isTouch === true) {
        const thresholdW =
          this.threshold < 1
            ? parseInt(this.itemW * this.threshold + "")
            : this.threshold;
        // 移动超过一定比例才滑动 否则回弹
        if (Math.abs(this.touchMoveX) > thresholdW) {
          if (this.touchMoveX > 0) {
            this.prev();
          } else {
            this.next();
          }
        } else {
          this.curr();
        }
      }
      // 手指离开时候重新开始定时器
      if (this.timer !== -1) clearInterval(this.timer);
      this.autoPlayFn();
    });
  }

  /** 指示器点击切换 */
  private handleIndicatorClick(index: number) {
    if (this.indicatorClick === false) return;
    this.redirectTo(index);
  }
}
</script>

<style scoped>
.swipe {
  position: relative;
  overflow: hidden;
}
.swipe .list,
.swipe .dots {
  margin: 0;
  padding: 0;
}
.swipe .list {
  display: flex;
  flex-wrap: nowrap;
  list-style: none;
}
.swipe .dots {
  position: absolute;
  left: 50%;
  bottom: 8px;
  transform: translateX(-50%);
}
.swipe .dots .dot {
  display: inline-block;
  width: 5px;
  height: 5px;
  margin-right: 4px;
  border-radius: 50%;
}
</style>
