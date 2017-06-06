<script>
  import {Line, mixins} from 'vue-chartjs'
  const { reactiveProp } = mixins

  export default Line.extend({
    mixins: [reactiveProp],
    data () {
      return {
        options: {
          scales: {
            yAxes: [{
              ticks: {
                beginAtZero: true
              },
              gridLines: {
                display: true
              }
            }],
            xAxes: [ {
              gridLines: {
                display: false
              }
            }]
          },
          legend: {
            display: true
          },
          responsive: true,
          maintainAspectRatio: false
        }
      }
    },
    mounted () {
      // this.chartData is created in the mixin
      this.renderChart(this.chartData, this.options)
    }
  })
</script>
