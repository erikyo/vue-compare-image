<template>

   <Radar
       :chart-options="chartOptions"
       :chart-data="chartData"
       :chart-id="chartId"
       :dataset-id-key="datasetIdKey"
       :plugins="plugins"
       :css-classes="cssClasses"
       :styles="styles"
       :width="width"
       :height="height"
   />

</template>


<script>
import { Radar } from 'vue-chartjs'

import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  PointElement,
  RadialLinearScale,
  LineElement,
  // Plugin
} from 'chart.js'

ChartJS.register(
    Title,
    Tooltip,
    Legend,
    PointElement,
    RadialLinearScale,
    LineElement
)

export default {
  name: "RadarChart",
  components: { Radar },
  props: {
    chartId: {
      type: String,
      default: 'radar-chart'
    },
    width: {
      type: Number,
      default: 600
    },
    height: {
      type: Number,
      default: 600
    },
    cssClasses: {
      default: '',
      type: String
    },
    styles: {
      type: Object,
      default: () => {}
    },
    plugins: {
      type: Array,
      default: () => []
    },
    radarData: {
      type: Object,
    },
    datasetIdKey: {
      default: 'name',
      type: String
    }
  },
  data() {
    return {
      result: {},
      temp: {},
      radarResults: [],
      chartData: {
        labels: [
          'sizebyte',
          'similarity',
          'noise',
          'encodetime',
          'cpu',
        ],
        datasets: [
          {
            label: this.radarData.left.name,
            borderColor: 'rgb(18,38,236)',
            pointBackgroundColor: 'rgb(78,89,164)',
            pointBorderColor: '#fff',
            pointHoverBackgroundColor: '#fff',
            pointHoverBorderColor: 'rgba(179,181,198)',
            data: [0,0,0,0,0]
          },
          {
            label: this.radarData.right.name,
            borderColor: 'rgb(255,224,99)',
            pointBackgroundColor: 'rgb(255,224,99)',
            pointBorderColor: '#fff',
            pointHoverBackgroundColor: '#fff',
            pointHoverBorderColor: 'rgb(255,224,99)',
            data: [0,0,0,0,0]
          }
        ]
      },
      chartOptions: {
        responsive: true,
        maintainAspectRatio: false,
        label: 0.2,
        elements: {
          line: {
            borderWidth: 3
          }
        },
        scales: {
          r: {
            ticks:{
              min : 0,
              max: 100,
              stepSize : 20,
              fontColor : "#ccc",
              backdropColor: '#222'
            },
          }
        }
      }
    }
  },
  methods: {
    renderChart() {
      this.renderChart(this.chartData, this.chartOptions);
    },
    calcPercentual(left, right) {
      if (left < right) {
        return {
          left: Math.round((left / right) * 100),
          right: 100
        }
      } else {
        return {
          left: 100,
          right: Math.round((right / left) * 100),
        }
      }

    },
    crossCalc() {
      let res = [] ;
      this.chartData.labels.forEach(metric => {
        res = res.concat(this.calcPercentual( this.radarData.left[metric], this.radarData.right[metric]))
      })
      return res
    },
    updateData() {

      this.result =this.crossCalc();

      this.chartData.datasets[0].data = Object.values(this.result).map( r => r.left );
      this.chartData.datasets[1].data = Object.values(this.result).map( r => r.right );
      this.chartData.datasets[0].label = this.radarData.left.name;
      this.chartData.datasets[1].label = this.radarData.right.name;
    }
  },
  watch: {
    radarData : {
      deep: true,
      handler: function() {
        this.$nextTick(() => {
          this.updateData();
        })
      }
    }
  },
}
</script>

<style scoped>

</style>