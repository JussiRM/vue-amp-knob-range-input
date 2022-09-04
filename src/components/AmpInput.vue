
<template>
	<div class="jussirm-amp-range-input" :class="['amp-range-input-theme-' + theme]">
		<div class="one-to-one-ratio-container">
			<div class="amp-container">
				<div class="center-point" ref="centerpoint"></div>
				<div class="handle-rotator" :style="rotatorStyle">
					<div class="handle"
						@mousedown.prevent="onHandleMouseDown"
					></div>
				</div>

				<div class="tick-indicator-container" v-for="tick in degreeTicks" :key="tick"
					:style="{ transform: `rotate(${tick}deg)` }"
				>
					<div></div>
				</div>
			</div>
		</div>
	</div>	
</template>

<script lang="ts">
import { defineComponent, PropType } from "vue"

interface Point {
	x: number,
	y: number
}

export default defineComponent({
	props: {
		modelValue: {
			type: Number as PropType<number>,
			required: true
		},

		count: {
			type: Number as PropType<number>,
			default: 10
		},

		unrotateAmount: {
			type: Number as PropType<number>,
			default: 50
		},

		theme: {
			type: String as PropType<string>,
			default: "default"
		}
	},

	data() { return {
		rotateAmount: 310,
		enableMouseMove: false
	}},

	watch: {
		modelValue() {
			this.rotateAmount = this.degreeTicks[this.modelValue];
		},

		unrotateAmount() {
			this.calculateRotateAmount({x: 0, y: 0});
		},

		count() {
			this.calculateRotateAmount({x: 0, y: 0});
		}
	},

	beforeMount() {
		this.rotateAmount = this.degreeTicks[this.modelValue];
	},

	mounted() {
		window.addEventListener("mousemove", this.onMouseMove);
	},

	computed: {
		// All the "ticks" that the handle locks on to
		degreeTicks() : number[] {
			let startDeg = (270 + this.unrotateAmount);

			let intervalDeg = (360 - (this.unrotateAmount*2)) / (this.count-1);

			let ticks = [startDeg];

			for(let i = 0; i < (this.count-1); i++) {
				let deg = startDeg + (intervalDeg*(i+1));
				if (deg > 360) {
					deg = (deg - 360);
				}
				ticks.push(deg);
			}

			return ticks;
		},

		rotatorStyle() : any {
			let rotateAngle = this.rotateAmount;

			let unrotateAmount = this.unrotateAmount;

			if (rotateAngle > (270 - unrotateAmount) && rotateAngle < 270) {
				rotateAngle = (270 - unrotateAmount);
			}
			else if (rotateAngle >= 270 && rotateAngle < (270 + unrotateAmount) ) {
				rotateAngle = (270 + unrotateAmount);
			}

			// Prevent rotator transition from jumping
			if (rotateAngle > 270) {
				rotateAngle = rotateAngle - 360;
			}

			return {
				'transform': 'rotate(' + rotateAngle + 'deg)'
			}
		}
	},

	methods: {
		onHandleMouseDown() {
			this.enableMouseMove = true;
			window.addEventListener("mouseup", this.onHandleMouseUp);
		},

		onHandleMouseUp() {
			this.enableMouseMove = false;
			window.removeEventListener("mouseup", this.onHandleMouseUp);
		},

		onMouseMove(ev:MouseEvent) {
			if (!this.enableMouseMove) {
				return;
			}

			const point = {
				x: ev.pageX,
				y: ev.pageY
			};

			this.calculateRotateAmount(point);
		},

		calculateRotateAmount(point:Point) {
			const elemCenterPoint = this.$refs.centerpoint as HTMLElement;
			if (!elemCenterPoint) {
				console.error("Can't resove element ref 'centerpoint'");
				return;
			}

			const p1 = point;

			const p2 = {
				x: elemCenterPoint.getBoundingClientRect().x,
				y: elemCenterPoint.getBoundingClientRect().y
			};

			// Calculate angle between point 1 and point 2 (cursor & center point element)
			let angleDeg = Math.atan2(p2.y - p1.y, p2.x - p1.x) * 180 / Math.PI;
			if (angleDeg < 0) {
				angleDeg = 360 - Math.abs(angleDeg);
			}
			
			// Snap into closest tick 
			angleDeg = this.getClosestTick(angleDeg);
			this.$emit('update:modelValue', this.degreeTicks.indexOf(angleDeg));

			this.rotateAmount = angleDeg;
		},

		getClosestTick(value:number) {
			let currentTick = -9999;
			this.degreeTicks.forEach(tick => {
				if (Math.abs(tick - value) < Math.abs(currentTick - value)) {
					currentTick = tick;
				}
			});
			return currentTick;
		}
	}
});
</script>

<style lang="scss">
	div.jussirm-amp-range-input {
		transform: scale(1.0);
		div.one-to-one-ratio-container {
			position: relative;
			width: 100%;
			padding-top: 100%;

			div.amp-container {
				position: absolute;
				top: 0;
				left: 0;
				bottom: 0;
				right: 0;
				border-radius: 50%;
				background-color: #363247;
				box-shadow: inset 0px 0px 20px 10px rgba(black, 0.33);

				div.center-point {
					position: absolute;
					z-index: 9999;
					top: 50%;
					left: 50%;
					width: 1px;
					height: 1px;
					transform: translate(-50%, -50%);
					background-color: green;
					opacity: 0;
				}

				div.handle-rotator {
					position: absolute;
					z-index: 1;
					left: 0;
					top: calc(50% - 5px);
					height: 10px;
					width: 100%;
					background-color: rgba(lighten(red, 15%), 0.0);
					//transform: rotate(00deg);
					transform-origin: center;
					//transition: transform 100ms ease;
					transition: all 150ms ease;

					div.handle {
						cursor: pointer;
						position: absolute;
						z-index: 2;
						left: 13%;
						top: 50%;
						height: 25px;
						width: 25px;
						border-radius: 50%;
						transform: rotate(0deg) translateY(-50%);
						background-color: #fff;
						box-shadow: inset 0px 0px 5px 1px rgba(0,0,0, 0.5);
					}
				}

				div.tick-indicator-container {
					position: absolute;
					left: 0;
					top: calc(50% - 3%);
					height: 6%;
					width: 100%;
					transform-origin: center;
				
					> div {
						position: absolute;
						background-color: rgba(#fff, 0.75);
						left: 0;
						top: 0;
						width: 10%;
						height: 100%;
						border-top-right-radius: 3px;
						border-bottom-right-radius: 3px;
					}
				}
			}
		}
	}
</style>