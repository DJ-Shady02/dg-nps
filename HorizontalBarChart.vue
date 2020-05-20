<script>
  import { mixins, HorizontalBar } from 'vue-chartjs';
  const { reactiveProp } = mixins;

  export default {
    extends: HorizontalBar,
    mixins: [reactiveProp],
    props: {
      chartData: Object,
      chartOptions: Object
    },
    mounted () {
      this.renderChart(this.chartData, this.chartOptions);
    }
  }
</script>
