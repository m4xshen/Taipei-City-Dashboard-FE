<!-- 在 <script setup> 中加入 computed 並修改 selectedDistrictSeries -->
<script setup>
import { ref, computed } from 'vue';

const props = defineProps(['chart_config', 'activeChart', 'series']);

const chartOptions = ref({
	chart: {
		stacked: true,
		toolbar: {
			show: false,
		},
	},
	colors: props.chart_config.color,
	grid: {
		show: false,
	},
	legend: {
		show: props.chart_config.categories ? true : false,
	},
	markers: {
		size: 3,
		strokeWidth: 0,
	},
	plotOptions: {
		radar: {
			size: 90,
			polygons: {
				connectorColors: '#444',
				strokeColors: '#555',
			},
		},
	},
	stroke: {
		show: true,
		width: 2,
	},
	tooltip: {
		custom: function ({ series, seriesIndex, dataPointIndex, w }) {
			// The class "chart-tooltip" could be edited in /assets/styles/chartStyles.css
			return '<div class="chart-tooltip">' +
				'<h6>' + w.globals.labels[dataPointIndex] + `${props.chart_config.categories ? '-' + w.globals.seriesNames[seriesIndex] : ''}` + '</h6>' +
				'<span>' + series[seriesIndex][dataPointIndex] + ` ${props.chart_config.unit}` + '</span>' +
				'</div>';
		},
	},
	xaxis: {
		categories: props.chart_config.categories ? props.chart_config.categories : [],
		labels: {
			offsetY: 5,
			formatter: function (value) {
				return value.length > 7 ? value.slice(0, 6) + "..." : value;
			}
		},
		type: 'category',
	},
	yaxis: {
		axisBorder: {
			color: '#000',
		},
		labels: {
			formatter: (value) => { return ''; },
		},
		// To fix a bug when there is more than 1 series
		// Orginal behavior: max will default to the max sum of each series
		max: function (max) {

			if (!props.chart_config.categories) {
				return max;
			}
			let adjustedMax = 0;
			props.series.forEach((element) => {
				const maxOfSeries = Math.max.apply(null, element.data);
				if (maxOfSeries > adjustedMax) {
					adjustedMax = maxOfSeries;
				}
			});
			return adjustedMax * 1.1;
		},
		
	},
});

const selectedDistrict = ref('松山區');

const selectedDistrictSeries = computed(() => {	
	const selectedDistrictSeries = props.series.filter((element) => {
		return element.name === selectedDistrict.value;
	});

	const ArrayOfSeries = props.series.map((element) => {

		return element.data;
	});

	const avgSeries = ArrayOfSeries.reduce((acc, cur) => {
		return acc.map((element, index) => {
			return element + cur[index];
		});
	}).map((element) => {
		return element / ArrayOfSeries.length;
	});


	
	return selectedDistrictSeries.concat({
		name: '平均',
		data: avgSeries,
	});
});
</script>

<template>
  <!-- 選單 -->
  <select v-if="activeChart === 'MultiRadarChart'" class="multiRadarChart-select" v-model="selectedDistrict">
    <option v-for="(seriesData, index) in series" :key="index" :value="seriesData.name">{{ seriesData.name }}</option>
  </select>
  <div v-if="activeChart === 'MultiRadarChart'">
    <!-- 將選擇的地區對應的 series 傳遞給 ApexChart -->
    <apexchart width="100%" height="270px" type="radar" :options="chartOptions" :series="selectedDistrictSeries"></apexchart>
  </div>
</template>

<style scoped>
.multiRadarChart-select {
  position: absolute;
  top: 0;
  right: 0;
  z-index: 1;
  width: 100px;
  height: 30px;
  border-radius: 5px;
  border: 1px solid #ccc;
  background-color: #888787;
}
</style>
