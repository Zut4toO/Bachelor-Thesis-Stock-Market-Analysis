<template>
  <div class="mt-8 flex flex-col w-[425px]">
    <div class="bg-white flex justify-center items-center w-[425px] h-[350px]">
      <Line
        id="my-chart-id"
        :options="chartOptions"
        :data="chartData"
        class=""
      />
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

    {{ console.log('predictor', predictor.predictorData) }}
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
    const predictor = inject('predict');
    return { predictor };
  },
  computed: {
    chartData() {
      if (!this.predictor.predictorData || !this.predictor.predictorName) {
        // Daten sind noch nicht verfügbar, gib ein leeres Objekt oder einen anderen Standardwert zurück
        return {
          labels: [],
          amountYears: 0,
          minY: 0,
          maxY: 100,
          datasets: [],
        };
      }

      //Time Series FX (Monthly) [1]
      //4. close
      //name:Meta Data [0] =>  2. From Symbol und 3. To Symbol

      //Time Series (Digital Currency Monthly) [1]
      //4b. close (USD)
      //name:Meta Data [0] =>  3. Digital Currency Name

      //data [0]
      //date oder value
      //name: name [2]

      //data [0]
      //date oder value
      //name: name [2]

      const dataArray = Object.values(this.predictor.predictorData);
      let getValues = dataArray[1];
      if (this.predictor.predictorName === 'fiat') {
        getValues = dataArray[1];
      } else if (this.predictor.predictorName === 'crypto') {
        getValues = dataArray[1];
      } else if (this.predictor.predictorName === 'kpi') {
        getValues = dataArray[3];
      } else if (this.predictor.predictorName === 'commodities') {
        getValues = dataArray[3];
      } else {
        getValues = dataArray[1];
      }

      //console.log('getValues', getValues);

      const monthlyTimeSeries = this.predictor.predictorData ? getValues : null;

      console.log('monthlyTimeSeries', monthlyTimeSeries);
      /*       console.log(
        'Object Keys',
        monthlyTimeSeries ? Object.keys(monthlyTimeSeries) : []
      ); */

      let labelsYears = [];
      let labelsMonth = [];

      if (
        this.predictor.predictorName === 'kpi' ||
        this.predictor.predictorName === 'commodities'
      ) {
        labelsYears = monthlyTimeSeries
          ? monthlyTimeSeries
              .map((item) => new Date(item.date).getFullYear().toString()) // Extract the year from the "date" property
              .filter((value, index, self) => self.indexOf(value) === index) // Remove duplicates
              .slice(0, 10) // Take the first 10 unique years
              .reverse() // Reverse the array to get the years in descending order
          : [];
        labelsMonth = monthlyTimeSeries
          ? monthlyTimeSeries
              .map((item) => new Date(item.date).toISOString().slice(0, 7))
              .filter((value, index, self) => self.indexOf(value) === index)
              .slice(0, 12)
              .reverse()
          : [];
      } else {
        labelsYears = monthlyTimeSeries
          ? Object.keys(monthlyTimeSeries)
              .map((date) => date.slice(0, 4))
              .filter((value, index, self) => self.indexOf(value) === index)
              .slice(0, 10)
              .reverse()
          : [];
        labelsMonth = monthlyTimeSeries
          ? Object.keys(monthlyTimeSeries)
              .map((date) => date.slice(0, 7))
              .filter((value, index, self) => self.indexOf(value) === index)
              .slice(0, 12)
              .reverse()
          : [];
      }

      const stockData = monthlyTimeSeries
        ? this.timeFilter === 'month'
          ? Object.values(monthlyTimeSeries)

              .map((stock) => {
                if (this.predictor.predictorName === 'fiat') {
                  return parseFloat(stock['4. close']).toFixed(2);
                } else if (this.predictor.predictorName === 'crypto') {
                  return parseFloat(stock['4b. close (USD)']).toFixed(2);
                } else if (
                  this.predictor.predictorName === 'kpi' ||
                  this.predictor.predictorName === 'commodities'
                ) {
                  return parseFloat(stock['value']).toFixed(2);
                } else {
                  return [];
                }
              })
              .slice(0, 12)
              .reverse()
          : labelsYears.map((year) => {
              const yearlyValues = Object.keys(monthlyTimeSeries).map(
                (stock) => {
                  if (year === stock.slice(0, 4)) {
                    return parseFloat(
                      monthlyTimeSeries[stock]['4b. close (USD)']
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
      //console.log('Filtered stockdata', stockData);

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
      console.log(labelsMonth);
      console.log(labelsYears);

      return {
        labels: this.timeFilter === 'month' ? labelsMonth : labelsYears,
        amountYears: labelsYears ?? 0,
        minY: minY,
        maxY: maxY,
        datasets: [
          {
            label: this.predictor.selectedPredictorName ?? '',
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
        aspectRatio: 1.075,
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
