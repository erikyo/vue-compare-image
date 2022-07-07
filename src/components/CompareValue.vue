<template>
  <div :class="'type-' + compareBy">
    <h5>{{ compareBy }}</h5>
    <h2>{{ ((result.win[compareBy] / result.lose[compareBy] * 100) - 100).toFixed(1) }}%</h2>
    <h4><span class="material-symbols-outlined"> arrow_upward </span> {{ result.win.formattedType }} {{ result.win.quality }} <br/>{{result.win[compareBy]}}</h4>
    <h4><span class="material-symbols-outlined"> arrow_downward </span> {{ result.lose.formattedType }} {{ result.lose.quality }} <br/>{{result.lose[compareBy]}}</h4>
  </div>
</template>

<script>
export default {
  name: "CompareValue",
  props: ['compareBy','winRule', 'images'],
  data: function() {
    return {
      result: {
        win: {},
        lose: {}
      }
    }
  },
  watch: {
    images : {
      deep: true,
      handler: function() {
        this.result = this.getResult()
      }
    }
  },
  methods: {
    getResult() {
      if (this.images.left && this.images.right)
        if ( this.images.left[this.compareBy] > this.images.right[this.compareBy] ) {
          return {win: this.images.left, lose: this.images.right}
        } else {
          return {win: this.images.right, lose: this.images.left}
        }
    }
  }
}
</script>