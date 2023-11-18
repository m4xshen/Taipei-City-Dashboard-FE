<!-- Developed by Taipei Urban Intelligence Center 2023 -->

<script setup>
import { ref } from 'vue';
import { useMapStore } from '../../store/mapStore';

const props = defineProps(['chart_config', 'activeChart', 'series', 'map_config']);
const mapStore = useMapStore();

const chartOptions = ref({
	chart: {
		stacked: true,
		toolbar: {
			show: false
		},
	},
	colors: props.chart_config.color,
	dataLabels: {
		enabled: props.chart_config.categories ? false : true,
		offsetY: 20,
	},
	grid: {
		show: false,
	},
	legend: {
		show: props.chart_config.categories ? true : false,

	},
	plotOptions: {
		bar: {
			borderRadius: 5,
		},
	},
	stroke: {
		colors: ['#282a2c'],
		show: true,
		width: 2,
	},
	tooltip: {
		// The class "chart-tooltip" could be edited in /assets/styles/chartStyles.css
		custom: function ({ series, seriesIndex, dataPointIndex, w }) {
			return '<div class="chart-tooltip">' +
				'<h6>' + w.globals.labels[dataPointIndex] + `${props.chart_config.categories ? '-' + w.globals.seriesNames[seriesIndex] : ''}` + '</h6>' +
				'<span>' + series[seriesIndex][dataPointIndex] + ` ${props.chart_config.unit}` + '</span>' +
				'</div>';
		},
	},
	xaxis: {
		axisBorder: {
			show: false,
		},
		axisTicks: {
			show: false,
		},
		categories: props.chart_config.categories ? props.chart_config.categories : [],
		labels: {
			offsetY: 5,
		},
		type: 'category',
	},
});

const selectedIndex = ref(null);
const selectedSeries = ref(null);

function handleDataSelection(e, chartContext, config) {
	if (!props.chart_config.map_filter) {
		return;
	}

	const {seriesIndex} = config;
	const {dataPointIndex} = config;

	const layer_id = "subsidy-circle";
	const property = "dist";
	const property2 = "service";
	const key = props.chart_config.map_filter[1][dataPointIndex];
	const key2 = props.chart_config.map_filter[3][seriesIndex];

	if (dataPointIndex !== selectedIndex.value || seriesIndex !== selectedSeries.value) {
		mapStore.addLayerFilter(layer_id, null, null, null, props.map_config,
			[
				"all",
				["==", ["get", property], key],
				["==", ["get", property2], key2]
			]
		);
		selectedIndex.value = dataPointIndex;
		selectedSeries.value = seriesIndex;
	} else {
		mapStore.clearLayerFilter(layer_id, status, props.map_config);
		selectedIndex.value = null;
		selectedSeries.value = null;
	}
}
</script>

<template>
	<div v-if="activeChart === 'ColumnChart'">
		<apexchart width="100%" height="270px" type="bar" :options="chartOptions" :series="series"
			@dataPointSelection="handleDataSelection"></apexchart>
	</div>
</template>
