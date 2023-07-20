<template>
  <div
    class="main-content min-h-screen bg-zinc-800 pt-6"
    @click="showList = false"
  >
    <div class="flex justify-center">
      <div>
        <ChartStocks />
        <ChartPredictor />
      </div>
      <div class="ml-8">
        <SelectCriterion class="mb-3" />
        <PredictorData class="mb-6" />
      </div>
    </div>
  </div>
</template>

<script>
import ChartStocks from './ChartStocks.vue';
import SelectCriterion from './SelectCriterion.vue';
import PredictorData from './PredictorData.vue';
import ChartPredictor from './ChartPredictor.vue';
export default {
  name: 'ContentComponent',
  components: { ChartStocks, SelectCriterion, PredictorData, ChartPredictor },
};
</script>

<style scoped></style>
