<!-- 在 <script setup> 中加入 computed 並修改 selectedDistrictSeries -->
<script setup>
import { ref, computed, onMounted, onUnmounted } from "vue";
import { useMapStore } from "../../store/mapStore";

const props = defineProps(["chart_config", "activeChart", "series"]);

const mapStore = useMapStore();

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

		offsetY: -20,
	},
	markers: {
		size: 3,
		strokeWidth: 0,
	},
	plotOptions: {
		radar: {
			size: 75,
			polygons: {
				connectorColors: "#444",
				strokeColors: "#555",
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
			return (
				'<div class="chart-tooltip">' +
				"<h6>" +
				w.globals.labels[dataPointIndex] +
				`${
					props.chart_config.categories
						? "-" + w.globals.seriesNames[seriesIndex]
						: ""
				}` +
				"</h6>" +
				"<span>" +
				series[seriesIndex][dataPointIndex] +
				` ${props.chart_config.unit}` +
				"</span>" +
				"</div>"
			);
		},
	},
	xaxis: {
		categories: props.chart_config.categories
			? props.chart_config.categories
			: [],
		labels: {
			offsetY: 5,
			formatter: function (value) {
				return value.length > 7 ? value.slice(0, 6) + "..." : value;
			},
		},
		type: "category",
	},
	yaxis: {
		axisBorder: {
			color: "#000",
		},
		labels: {
			formatter: (value) => {
				return "";
			},
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

const selectedDistrict = ref("松山區");

const selectedDistrictSeries = computed(() => {
	const selectedDistrictSeries = props.series.filter((element) => {
		return element.name === selectedDistrict.value;
	});
	mapStore.clearLayerFilter(`child_care_institution-symbol`);
	mapStore.addLayerFilter(
		`child_care_institution-symbol`,
		"district",
		selectedDistrictSeries[0].name,
	);

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
		name: "平均",
		data: avgSeries,
	});
});

const dropdownOpen = ref(false);
const toggleDropdown = () => {
	setTimeout(() => {
		dropdownOpen.value = !dropdownOpen.value;
	}, 50);
};

const selectOption = (option) => {
	selectedDistrict.value = option;
	dropdownOpen.value = false;
};
const closeDropdown = () => {
	if (dropdownOpen.value === true) {
		dropdownOpen.value = false;
	}
};

onMounted(() => {
	document.addEventListener("click", closeDropdown);
});

onUnmounted(() => {
	document.removeEventListener("click", closeDropdown);
});
</script>

<template>
	<!-- 選單 -->
	<div :style="{ marginTop: '20px' }">
		<div class="custom-select" v-if="activeChart === 'MultiRadarChart'">
			<div class="select-head" @click="toggleDropdown">
				<span class="select-head-cont">{{ selectedDistrict }}</span>
				<span class="select-icon"
					><img src="/public/images/map/select.png"
				/></span>
			</div>
			<ul v-if="dropdownOpen" class="options">
				<li
					v-for="(seriesData, index) in series"
					:key="index"
					@click="selectOption(seriesData.name)"
					class="option-item"
				>
					{{ seriesData.name }}
				</li>
			</ul>
		</div>

		<div v-if="activeChart === 'MultiRadarChart'" class="MultiRadarChart">
			<!-- 將選擇的地區對應的 series 傳遞給 ApexChart -->
			<apexchart
				width="100%"
				height="270px"
				type="radar"
				:options="chartOptions"
				:series="selectedDistrictSeries"
			></apexchart>
		</div>
	</div>
</template>

<style scoped lang="scss">
.MultiRadarChart {
	position: relative;
	z-index: 1;
}
.multiRadarChart-select {
	position: absolute;
	top: 0;
	right: 0;
	z-index: 1;
	width: 100px;
	height: 30px;
	border-radius: 5px;
	border: 1px solid #ccc;
	background-color: transparent;
	:focus {
		border: 1px solid #ccc;
	}
	:hover::after {
		color: #f39c12;
	}
	option {
		background-color: transparent;
		color: #333;
	}
}
.custom-select {
	position: absolute;
	display: inline-block;
	z-index: 2;
}

.select-head {
	padding: 10px;
	border: 1px solid #ccc;
	border-radius: 5px;
	cursor: pointer;
	font-size: 12px;
	gap: 8px;
}
.select-icon {
	margin: 8px;
}
.options {
	position: relative;
	max-height: 200px;
	overflow-y: auto;
	z-index: 2;
	top: 1px;
	left: 0;
	width: 100%;
	border-bottom: 1px solid #ccc;
	border-radius: 0 0 5px 5px;
	list-style-type: none; // 移除點點符號
	padding: 0;
	margin: 0;
	background-color: #333;
	/* 滚动条样式 */
	scrollbar-color: #ccc white; /* Firefox 支持的滚动条样式，颜色自定义 */
	scrollbar-width: thin; /* Firefox 支持的滚动条宽度，可选值为 "auto"、"thin"、"none" */

	&::-webkit-scrollbar {
		width: 8px; /* WebKit (Chrome, Safari, Edge) 支持的滚动条宽度 */
	}

	&::-webkit-scrollbar-thumb {
		background-color: #565656; /* 滚动条拖动部分颜色 */
		border-radius: 4px; /* 滚动条拖动部分圆角 */
	}

	// &::-webkit-scrollbar-track {
	// 	background-color: white; /* 滚动条轨道部分颜色 */
	// 	border-radius: 5px; /* 滚动条轨道部分圆角 */
	// 	box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.1); /* 滚动条轨道部分阴影 */
	// }
}

.option-item {
	font-size: 12px;
	padding: 8px;
	cursor: pointer;
	transition: background-color 0.3s;
}

.option-item:hover {
	background-color: #5a9cf8;
}
</style>
