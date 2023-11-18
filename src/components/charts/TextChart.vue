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
</script>

<template>
	<div v-if="activeChart === 'TextChart'" class="TextChart">
		<div
			:style="{
				display: 'flex',
				flexDirection: 'column',
				gap: '20px',
				margin: '15% 10%',
				width: '80%',
			}"
		>
			<div
				:style="{
					display: 'flex',
					flexDirection: 'column',
					fontSize: '14px',
					gap: '4px',
				}"
			>
				商圈
				<div class="square">
					<div
						:style="{
							display: 'flex',
							flexDirection: 'row',
							alignItems: 'center',
						}"
					>
						<div
							class="circle"
							:style="{ backgroundColor: '#F65658' }"
						></div>

						吃
					</div>
					<div
						:style="{
							display: 'flex',
							flexDirection: 'row',
							alignItems: 'center',
						}"
					>
						<div
							class="circle"
							:style="{ backgroundColor: '#F49F36' }"
						></div>

						買
					</div>
					<div
						:style="{
							display: 'flex',
							flexDirection: 'row',
							alignItems: 'center',
						}"
					>
						<div
							class="circle"
							:style="{ backgroundColor: '#2F8AB1' }"
						></div>

						玩
					</div>
				</div>
			</div>
			<div
				:style="{
					display: 'flex',
					flexDirection: 'column',
					fontSize: '14px',
					gap: '4px',
				}"
			>
				其他
				<div
					class="square"
					:style="{
						justifyContent: 'flex-start',
					}"
				>
					<div
						:style="{
							display: 'flex',
							flexDirection: 'row',
							alignItems: 'center',
							gap: '4px',
						}"
					>
						<img
							src="/public/images/map/breastfeeding.png"
							width="20"
						/>

						哺乳室
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<style scoped lang="scss">
.TextChart {
	align-self: center;
	width: 100%;
	gap: 10px;
	display: flex;
	flex-direction: column;
	max-height: 100%;
}

.circle {
	border-radius: 50%;
	width: 30px;
	height: 30px;
	background-color: #bcbcbc;
	margin-right: 4px;
	&.selected {
		background-color: #333;
	}
}
.square {
	border-radius: 10px;
	width: 80%;
	padding: 5%;
	border: 1px solid #bcbcbc;
	display: flex;
	flex-direction: row;
	justify-content: space-between;
	align-items: center;
	font-size: 14px;
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
