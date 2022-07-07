<template>
  <div id="split-compare-wrap" >

    <header>
      <h2>Source Image: <b>{{ originalImage.filename }}</b></h2>
      <p><span class="material-symbols-outlined">compress</span> {{originalImage.compression}} <span class="material-symbols-outlined">palette</span> {{originalImage.channels}} <span class="material-symbols-outlined">folder_zip</span> {{originalImage.filesize}}</p>
      <p><span class="material-symbols-outlined">aspect_ratio</span> {{originalImage.width}} x {{originalImage.height}}</p>
      <p><span class="material-symbols-outlined">warning</span> SSIM {{originalImage.similarity}} - PSNR {{ originalImage.psnr ? originalImage.psnr.all : 'no data' }}</p>

      <h4>Reference Image: <b>{{ referenceImage.filename }}</b></h4>
      <p><span class="material-symbols-outlined">compress</span> {{referenceImage.compression}} <span class="material-symbols-outlined">palette</span> {{referenceImage.channels}} <span class="material-symbols-outlined">folder_zip</span> {{referenceImage.filesize}}</p>
      <p><span class="material-symbols-outlined">aspect_ratio</span> {{referenceImage.width}} x {{referenceImage.height}}</p>
    </header>

    <div id="split-ui" :style="{ width: width + 'px' }">

      <data-parser ref="parser"></data-parser>

      <split-compare :style="{ width: width + 'px' , height: height + 'px' }">
        <template v-slot:left>
          <img v-bind:src="path + currentCompare.left.filename" />
        </template>
        <template v-slot:right>
          <img v-bind:src="path + currentCompare.right.filename" />
        </template>
      </split-compare>

      <div class="image-select">
        <select v-model="currentCompare.left.filename" @change="onChange(currentCompare.left.filename,'left')">
          <option v-for="(image, filename) in imagesData" :value="filename" v-bind:key="filename">
            {{ extendedFormatName(image) }}
          </option>
        </select>
        <p class="vs">VS</p>
        <select v-model="currentCompare.right.filename" @change="onChange(currentCompare.right.filename,'right')">
          <option v-for="(image, filename) in imagesData" :value="filename" v-bind:key="filename">
            {{ extendedFormatName(image) }}
          </option>
        </select>
      </div>

      <div class="flex-content result-row">
        <compare-value compare-by="sizebyte" win-rule="lower" :images="currentCompare"></compare-value>
        <compare-value compare-by="encodetime" win-rule="lower" :images="currentCompare"></compare-value>
        <compare-value compare-by="similarity" win-rule="higher" :images="currentCompare"></compare-value>
      </div>

      <RadarChart :radarData="chartData" :height="600" />

    </div>

    <compare-table :tableImagesData="imagesData"></compare-table>

    <pre>{{currentCompare.left}} vs {{currentCompare.right}}</pre>

  </div>

</template>

<script>
import DataParser from './components/DataParser.vue'
import SplitCompare from './components/SplitCompare.vue'
import CompareValue from './components/CompareValue.vue'
import CompareTable from './components/CompareTable.vue'
import RadarChart from './components/RadarChart.vue'

export default {
  name: 'App',
  components: {
    RadarChart,
    DataParser,
    SplitCompare,
    CompareValue,
    CompareTable,
  },
  data() {
      return {
        imagesData: {},
        width: null,
        height: null,
        path: '',
        originalImage: {},
        referenceImage: {},
        currentCompare: {
          left: { filename: '' },
          right: { filename: '' },
        },
      }
    },
    methods: {
      onChange(value, target){
        this.currentCompare[target] = this.imagesData[value]
        this.$emit('getResult', this.currentCompare)
      },
      humanFileSize(size) {
        var i = Math.floor(Math.log(size) / Math.log(1024));
        return (size / Math.pow(1024, i)).toFixed(2) * 1 + ' ' + ['B', 'kB', 'MB', 'GB', 'TB'][i];
      },
      extendedFormatName(image) {
        return `${image.formattedType + " " + (image.quality ? "(" + image.quality + "%)" : '')}`
      },
    },
  watch: {
    imagesData: {
      handler: function() {
        this.$emit('updateTable', this.imagesData)
      }
    }
  },
  computed: {
    chartData() { return this.currentCompare }
  }
}
</script>

<style lang="scss">

:root {
  --color-dark: #222;
  --color-light: #F3F3F3;
  --color-primary: #03A9F4;
  --divider-thickness: 4px;
  --divider-handle--size: 32px;
}

// css reset
html {
  line-height: 1.15;
  -webkit-text-size-adjust: 100%;
}
body {
  font-family: Open Sans, sans-serif;
  clear: both;
  margin: 0;
  font-weight: 300;
}
main {
  display: block;
}
img {
  border-style: none;
  height: auto;
  max-width: 100%
}
pre {
  font-family: monospace, monospace;
  font-size: 1em;
}
h1, h2, h3, h4, h5, h6, p {
  margin: 0;
}
b,
strong {
  font-weight: bolder;
}
button,
input,
optgroup,
select,
textarea {
  font-family: inherit;
  font-size: 100%;
  line-height: 1.15;
  margin: 0;
}

// app style
#split-compare-wrap {
  background: var(--color-dark);
  padding: 16px;

  div#split-compare-wrap * {
    max-width: 100%;
  }

  // ui Elements
  h1,
  h2,
  h3,
  h4,
  p {
    color: var(--color-light);
    margin: 0 0 1em;
  }

  select {
    position: relative;
    min-width: 240px;
    font-size: 20px;
    background: var(--color-light);
    border: 1px solid var(--color-primary);
    box-shadow: 0 3px 5px -2px var(--color-dark), 0 20px 30px -6px var(--color-dark);
    border-radius: 6px;
  }

  // Comparator wrapper style
  #split-ui {
    display: flex;
    width: 800px;
    flex-direction: column;
    margin: auto;
  }

  .compare-wrapper {
    display: flex;
    justify-content: center;
    margin: auto;
    overflow: hidden;
    padding: 0;
    border: 0;
    box-sizing: border-box;
    position: relative;
    transform: scale(1);
    width: 100%;
  }

  header {
    text-align: center;
    margin: 16px 0 32px;

    h1, h2, h3, h4 {
      margin: 16px 0 0;
      font-weight: 300;
    }

    p {
      font-size: 14px;
      line-height: 2.2;
      margin: 0;
      color: #bbb;
    }

    .material-symbols-outlined {
      vertical-align: middle;
      margin: 0 2px 0 16px;
      color: #ff9800;
      opacity: .8;
    }
  }

  .image-select {
    display: flex;
    justify-content: space-between;
    padding: 24px 8px;
    line-height: 48px;
    font-weight: 600;

    select {
      max-width: 33%;
    }

    .vs {
      font-size: 48px;
      line-height: 48px;
      margin: 16px 0;
    }
  }

  .flex-content {
    display: flex;
    justify-content: space-between;
    width: 100%;
    overflow: hidden;
  }

  .result-row {
    text-align: center;
  }
}


@media screen and (max-width: 782px) {
  #split-compare-wrap #split-ui,
  #split-compare-wrap .compare-wrapper {
    max-width: 100%;
    height: auto !important;
  }
}

.split-handle {
  appearance: none;
  background: none;
  border: 0;
  box-sizing: border-box;
  cursor: pointer;
  height: 100%;
  left: 0;
  margin: 0;
  padding: 0;
  outline: 0;
  position: fixed;
  top: 0;
  width: 100%;
  z-index: 1;

  &:active {
    cursor: col-resize;
  }

  &::-moz-range-thumb,
  &::-webkit-slider-thumb {
    -moz-appearance: none;
    -webkit-appearance: none;
    appearance: none;
    background: none;
    border: 0;
    border-radius: 50%;
    cursor: col-resize;
    height: var(--divider-handle--size);
    width: var(--divider-handle--size);
  }
}

.split-overlay {
  height: 100%;
  position: absolute;
  pointer-events: none;
  top: 0;
  left: 0;
}

.split-divider {
  border-right: var(--divider-thickness) solid var(--color-dark);
  height: 100%;
  position: absolute;
  z-index: 2;
  right: calc(var(--divider-thickness) / 2 * -1);
  width: var(--divider-thickness);
}


.split-icon {
  &:before {
    background-image: url('data:image/svg+xml,%3Csvg xmlns="http://www.w3.org/2000/svg" height="30" width="42" viewBox="0 0 42 30"%3E%3Cpath fill="rgb(200, 200, 200)" d="m 0 15 l 10 10 l 0 -20 z m 42 0 l -10 10 l 0 -20 z"/%3E%3C/svg%3E');
    background-position: center;
    background-repeat: no-repeat;
    background-size: 50%;
    z-index: 4;
  }
  &:after {
    background: var(--color-dark);
    border: var(--divider-thickness) solid var(--color-dark);
    z-index: 3;
  }
  &:before,
  &:after {
    border-radius: 50%;
    box-sizing: border-box;
    content: "";
    display: block;
    height: var(--divider-handle--size);
    position: absolute;
    right: calc((var(--divider-handle--size) / 2) * -1);
    top: calc(50% - (var(--divider-handle--size) / 1.8));
    width: var(--divider-handle--size);
  }
}


#radar-chart {
  margin: 64px auto;
}
</style>
