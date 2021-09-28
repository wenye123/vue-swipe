<template>
  <li class="swipe-item">
    <slot></slot>
  </li>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";

@Component({
  components: {},
})
export default class SwipeItem extends Vue {}
</script>

<style scoped>
.swipe-item img {
  width: 100%;
}
</style>
