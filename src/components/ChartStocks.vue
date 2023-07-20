<template>
  <div>
    <div class="w-[850px] h-[400px] bg-white flex justify-center">
      <Line id="my-chart-id" :options="chartOptions" :data="chartData" />
    </div>
    <div v-if="yearsLength" class="text-white text-sm flex justify-end">
      Graph eingrenzen:
      <button
        :class="
          'font-semibold ml-1 mr-2 ' +
          (timeFilter === 'month' && 'text-red-500')
        "
        @click="timeFilter = 'month'"
      >
        Letzten 12 Monate
      </button>
      <button
        :class="
          'font-semibold ml-1 mr-2 ' + (timeFilter === 'year' && 'text-red-500')
        "
        @click="timeFilter = 'year'"
      >
        Letzten {{ yearsLength }} Jahre
      </button>
    </div>
  </div>
</template>

<script>
import { Line } from 'vue-chartjs';
import { inject } from 'vue';
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend,
} from 'chart.js';

ChartJS.register(
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend
);

export default {
  name: 'ChartStocks',
  components: { Line },
  setup() {
    const sharedStocks = inject('stocks');
    //console.log('sharedStocks', sharedStocks.stockData);
    return { sharedStocks };
  },
  computed: {
    chartData() {
      const monthlyTimeSeries = this.sharedStocks.stockData
        ? this.sharedStocks.stockData['Monthly Adjusted Time Series']
        : null;

      const labelsYears = monthlyTimeSeries
        ? Object.keys(monthlyTimeSeries)
            .map((date) => date.slice(0, 4))
            .filter((value, index, self) => self.indexOf(value) === index)
            .slice(0, 10)
            .reverse()
        : [];
      const labelsMonth = monthlyTimeSeries
        ? Object.keys(monthlyTimeSeries)
            .map((date) => date.slice(0, 7))
            .filter((value, index, self) => self.indexOf(value) === index)
            .slice(0, 12)
            .reverse()
        : [];

      const stockData = monthlyTimeSeries
        ? this.timeFilter === 'month'
          ? Object.values(monthlyTimeSeries)

              .map((stock) => {
                return parseFloat(stock['5. adjusted close']).toFixed(2);
              })
              .slice(0, 12)
              .reverse()
          : labelsYears.map((year) => {
              const yearlyValues = Object.keys(monthlyTimeSeries).map(
                (stock) => {
                  if (year === stock.slice(0, 4)) {
                    return parseFloat(
                      monthlyTimeSeries[stock]['5. adjusted close']
                    ).toFixed(2);
                  }
                }
              );
              const filteredYearlyValues = yearlyValues.filter((value) => {
                return !isNaN(value);
              });

              const sum = filteredYearlyValues.reduce((accumulator, value) => {
                return accumulator + parseFloat(value);
              }, 0);

              return (sum / filteredYearlyValues.length).toFixed(2);
            })
        : [];
      //console.log('Used years', labelsYears);
      //console.log('getRequest', monthlyTimeSeries);
      console.log('Filtered stockdata', stockData);

      const minY = Math.max(
        0,
        Math.floor(
          Math.min(...stockData.map((stock) => parseInt(stock) - 10)) / 10
        ) * 10
      );
      const maxY =
        Math.ceil(
          Math.max(...stockData.map((stock) => parseInt(stock) + 10)) / 10
        ) * 10;

      //console.log(minY, maxY);
      //console.log(labelsMonth);
      //console.log(labelsYears);

      return {
        labels: this.timeFilter === 'month' ? labelsMonth : labelsYears,
        amountYears: labelsYears ?? 0,
        minY: minY,
        maxY: maxY,
        datasets: [
          {
            label: this.sharedStocks.stockName?.['2. name'] ?? '',
            data: stockData ?? [],
            backgroundColor: '#f87979',
            borderColor: '#f87979',
          },
        ],
      };
    },
    chartOptions() {
      return {
        responsive: true,
        scales: {
          y: {
            min: this.chartData.minY ?? 0,
            max: this.chartData.maxY ?? 100,
          },
        },
      };
    },
    yearsLength() {
      const amountYears = this.chartData.amountYears;
      return amountYears?.length ?? 0;
    },
  },
  data() {
    return {
      timeFilter: 'month',
    };
  },
};
</script>
