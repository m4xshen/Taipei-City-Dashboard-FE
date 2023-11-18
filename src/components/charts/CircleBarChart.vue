<!-- Developed by Taipei Urban Intelligence Center 2023 -->

<script setup>
import { ref } from "vue";
import { useMapStore } from "../../store/mapStore";

const props = defineProps([
	"chart_config",
	"activeChart",
	"series",
	"map_config",
	"chartData",
]);
const mapStore = useMapStore();

const chartData = props.series[0].data;

const selectedIndex = ref(null);
const isClick = ref(Array(chartData.length).fill(false));
function handleDataSelection(e, chartContext, config) {
	if (!props.chart_config.map_filter) {
		return;
	}
	// 取消先前選中的bar
	if (selectedIndex.value !== null) {
		isClick.value[selectedIndex.value] = false;
	}
	isClick.value[config.dataPointIndex] =
		!isClick.value[config.dataPointIndex];

	if (config.dataPointIndex !== selectedIndex.value) {
		isClick.value[config.dataPointIndex] = true;
		mapStore.addLayerFilter(
			`${props.map_config[0].index}-${props.map_config[0].type}`, //圖層ID
			"c_name", //欄位名稱
			props.chart_config.map_filter[1][config.dataPointIndex], //欄位值
			"district",
		);
		selectedIndex.value = config.dataPointIndex;
	} else {
		isClick.value[config.dataPointIndex] = false;
		mapStore.clearLayerFilter(
			`${props.map_config[0].index}-${props.map_config[0].type}`,
			"district",
		);
		selectedIndex.value = null;
	}
}
</script>

<template>
	<div v-if="activeChart === 'CircleBarChart'" class="CircleBarChart">
		<div class="unit">商圈數量 單位：個</div>
		<div
			v-for="(item, index) in chartData"
			:key="index"
			class="bar"
			:class="{ selected: isClick[index] }"
			@click="
				{
					handleDataSelection($event, chartData, {
						dataPointIndex: index,
					});
				}
			"
		>
			<div
				:style="{
					fontSize: '10px',
					marginRight: '8px',
					color: white,
				}"
				class="font"
				:class="{ selected: isClick[index] }"
			>
				{{ item.x }}
			</div>
			<div
				v-for="circleIndex in item.y"
				:key="circleIndex"
				class="circle"
				:class="{ selected: isClick[index] }"
			></div>
			<div
				:style="{
					display: 'flex',
					flexDirection: 'row',
					justifyContent: 'flex-end',
					alignItems: 'center',
					position: 'absolute',
					right: '8px',
				}"
			>
				<div
					:style="{
						fontSize: '16px',
						marginRight: '8px',
						color: white,
					}"
					:class="{ selected: isClick[index] }"
					class="font"
				>
					{{ item.y }}
				</div>
			</div>
		</div>
	</div>
</template>

<style scoped lang="scss">
.CircleBarChart {
	align-self: center;
	width: 100%;
	gap: 1px;
	// max-height: 100%;
	display: flex;
	flex-direction: column;
	align-items: flex-start;
	justify-content: flex-start;
	margin-top: 30px;
	box-sizing: border-box;
}
.unit {
	font-size: 12px;
	color: #bababa;
}
.font {
	color: white;
	&.selected {
		color: #333;
	}
}
.bar {
	width: 80%;
	display: flex;
	justify-content: flex-start;
	align-items: center;
	flex-direction: row;
	position: relative;
	padding: 8px;
	box-sizing: border-box;

	&:hover {
		cursor: pointer;
		background: #5b70e8;
		border-radius: 4px;
		color: white;
		.circle {
			background: white;
		}
	}
	&.selected {
		border-radius: 4px;
		background: #89d2dc;
		&:hover {
			.circle {
				background: #333;
			}
		}
	}

	&.fade-in {
		animation: ease-in 0.5s ease-in-out forwards;
	}
}
.circle {
	border-radius: 50%;
	width: 6px;
	height: 6px;
	background-color: #f65658;
	margin-right: 4px;
	&.selected {
		background-color: #333;
	}
}
.label {
	flex-direction: column;
	display: flex;
	justify-content: center;
	align-items: center;
	&:hover {
		cursor: pointer;
		filter: drop-shadow(4px 4px 20px rgba(255, 255, 255, 0.9));
		translate: scale(5);
	}
	&.selected {
		color: #333;
	}
}
@keyframes ease-in {
	0% {
		opacity: 0;
	}

	100% {
		opacity: 1;
	}
}

@for $i from 1 through 5 {
	.initial-animation-#{$i} {
		animation-name: ease-in;
		animation-duration: 0.2s;
		animation-delay: 0.05s * ($i - 1);
		animation-timing-function: linear;
		animation-fill-mode: forwards;
		opacity: 0;
	}
}
</style>
