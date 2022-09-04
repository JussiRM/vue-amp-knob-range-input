<template>
	<div style="margin: 0 auto; max-width: 600px;">
		<div class="d-flex mb-2">
			<div class="pe-2">Container width: {{formatNum(width, 3)}}px</div>
			<input type="range" v-model.number="width" :min="25" :max="300" step="5" />
		</div>
		
		<div class="d-flex mb-2">
			<div class="pe-2">V-model value: {{formatNum(value)}}</div>
			<input type="range" v-model.number="value" :min="0" :max="ticks" />
		</div>

		<div class="d-flex mb-2">
			<div class="pe-2">Tick count: {{formatNum(ticks)}}</div>
			<input type="range" v-model.number="ticks" :min="3" :max="30" />
		</div>

		<div class="d-flex mb-2">
			<div class="pe-2">Unrotate amount (deg): {{formatNum(unrotateAmount, 3)}}</div>
			<input type="range" v-model.number="unrotateAmount" :min="0" :max="120" />
		</div>

		<div class="d-flex mb-2">
			<div class="pe-2">Theme: </div>
			<select v-model="theme">
				<option value="default">Default</option>
				<option value="marshall">Marshall</option>
			</select>
		</div>

		<div style="border: 1px dashed gray; padding: 20px;" :style="{width: width + 'px'}">
			<AmpInput 
				:count="ticks" 
				v-model="value" 
				:unrotate-amount="unrotateAmount" 
				:theme="theme"
			/>
		</div>
	</div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import HelloWorld from "./components/HelloWorld.vue";
import AmpInput from "@/components/AmpInput.vue";

export default defineComponent({
	name: "App",
	components: {
		AmpInput,
	},

	data() {
		return {
			width: 200,
			value: 4,
			ticks: 10,
			unrotateAmount: 50,
			theme: "default"
		};
	},

	methods: {
		formatNum(num:number, length = 2) {
			let str = num.toString();
			
			if (str.length < length) {
				str = "0".repeat(length - str.length) + str;
			}

			return str;
		}
	}
});
</script>

<style lang="scss">
@import "./themes/marshall.scss";

#app {
	font-family: Avenir, Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	color: #2c3e50;
	margin-top: 60px;
}
</style>
