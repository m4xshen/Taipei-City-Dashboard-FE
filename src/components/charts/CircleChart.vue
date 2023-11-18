<!-- Developed by Taipei Urban Intelligence Center 2023 -->

<script setup>
import { ref, onMounted } from "vue";
import { useMapStore } from "../../store/mapStore";

const props = defineProps([
	"chart_config",
	"activeChart",
	"series",
	"map_config",
	"chartData",
]);
const mapStore = useMapStore();
const currentIndex = ref(-1);
function startAnimation() {
	currentIndex.value = -1;
	const intervalId = setInterval(() => {
		currentIndex.value += 1;
		if (currentIndex.value === chartData.length - 1) {
			clearInterval(intervalId);
		}
	}, 200);
}
onMounted(() => {
	startAnimation();
});
const chartData = props.series[1].data;
const circlePositions = {
	R: { x: 30, y: 30 },
	G: { x: 150, y: 30 },
	BL: { x: 10, y: 145 },
	BR: { x: 110, y: 140 },
	O: { x: 220, y: 110 },
	Y: { x: 190, y: 190 },
};
const numToColor = {
	R: "#E3002C",
	G: "#0E693E",
	BL: "#2476BF",
	BR: "#9A6940",
	O: "#F99526",
	Y: "#fcd100",
};
const numToMRT = {
	R: "淡水信義線",
	G: "松山新店線",
	BL: "板南線",
	BR: "文湖線",
	O: "中和新蘆線",
	Y: "環狀線",
};
const maxNumber = Math.max(...chartData.map((item) => item.y));
const selectedIndex = ref(null);
const isClick = ref(Array(chartData.length).fill(false));
const status = ref("MRT");

function handleDataSelection(e, chartContext, config) {
	if (!props.chart_config.map_filter) {
		return;
	}
	// 取消先前選中的bar
	if (selectedIndex.value !== null) {
		isClick.value[selectedIndex.value] = false;
	}

	let fieldValue;
	switch (config.dataPointIndex) {
		case 0:
			fieldValue = "R";
			break;
		case 1:
			fieldValue = "BR";
			break;
		case 2:
			fieldValue = "BL";
			break;
		case 3:
			fieldValue = "G";
			break;
		case 4:
			fieldValue = "O";
			break;
		case 5:
			fieldValue = "Y";
			break;
		default:
			break;
	}
	if (config.dataPointIndex !== selectedIndex.value) {
		isClick.value[config.dataPointIndex] = true;
		mapStore.addLayerFilter(
			`${props.map_config[0].index}-${props.map_config[0].type}`, //圖層ID
			"c_MRT", //欄位名稱
			fieldValue, //欄位值
			"MRT",
		);
		selectedIndex.value = config.dataPointIndex;
	} else {
		isClick.value[config.dataPointIndex] = false;
		mapStore.clearLayerFilter(
			`${props.map_config[0].index}-${props.map_config[0].type}`,
			"MRT",
		);
		selectedIndex.value = null;
	}
}
</script>

<template>
	<div v-if="activeChart === 'CircleChart'" class="circlechart">
		<transition-group name="fade" tag="div">
			<div
				v-for="(item, index) in chartData"
				:key="index"
				class="circle"
				:class="{ selected: isClick[index] }"
				v-show="index <= currentIndex"
				:style="{
					position: 'absolute',
					backgroundColor: numToColor[item.x] || '#000',
					width: `${(item.y / maxNumber) * 50 + 65}px`,
					height: `${(item.y / maxNumber) * 50 + 65}px`,
					animation: `fadeIn ${index * 0.2}s ease-in-out forwards`,
					top: `${circlePositions[item.x].y}px`,
					left: `${circlePositions[item.x].x}px`,
				}"
				@click="
					{
						handleDataSelection($event, chartData, {
							dataPointIndex: index,
						});
					}
				"
			>
				<div class="label">
					<div :style="{}">{{ item.x }}</div>
					<div :style="{ fontSize: '10px' }">
						{{ numToMRT[item.x] }}
					</div>
					<div
						:style="{
							flexDirection: 'row',
							display: 'flex',
							alignItems: 'flex-end',
							fontSize: '24px',
						}"
					>
						{{ item.y }}
						<div :style="{ fontSize: '8px' }">個</div>
					</div>
				</div>
			</div>
		</transition-group>
	</div>
</template>

<style scoped lang="scss">
.circlechart {
	width: 100%;
}
.circle {
	border-radius: 50%;
	flex-direction: column;
	display: flex;
	justify-content: center;
	align-items: center;
	&:hover {
		cursor: pointer;
		filter: drop-shadow(4px 4px 20px rgba(255, 255, 255, 0.1));
		translate: scale(5);
	}
	&.selected {
		filter: brightness(0.8);
	}
}
.label {
	flex-direction: column;
	display: flex;
	justify-content: center;
	align-items: center;
}
@keyframes ease-in {
	0% {
		opacity: 0;
	}

	100% {
		opacity: 1;
	}
}
</style>
