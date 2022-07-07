<template>
  <div class="compare-wrapper">

    <input type="range" step="0.2" class="split-handle" v-model="currentValue">

    <div class="split-overlay" :style="{ width: currentValue + '%' }">
      <div class="split-icon"></div>
      <div class="split-divider"></div>
    </div>

    <div class="split-image split-left">
      <slot name="left"></slot>
    </div>

    <div class="split-image split-right" :style="{clipPath: 'inset(0 0 0 ' + currentValue + '%)'}">
      <slot name="right"></slot>
    </div>

  </div>
</template>

<script>
export default {
  name: "splitCompare",
  data(){
    return {
      currentValue: 50,
    }
  },
}
</script>

<style lang="scss">
.split-image {
  > img {
    height: 100%;
    width: 100%;
  }
  &.split-left {
    position: relative;
    width: 100%;
  }
  &.split-right {
    height: 100%;
    position: absolute;
    right: 0;
    top: 0;
    width: 100%;
  }
}
</style>