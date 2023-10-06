<script lang="ts">
	import { createEventDispatcher } from 'svelte';

	export let value = 50;
	export let min = 0;
	export let max = 100;
	export let step = 1;

	const dispatch = createEventDispatcher();

	function onChange() {
		dispatch('change', { value });
	}
</script>

<div>
	<input
		type="range"
		bind:value
		{min}
		{max}
		{step}
		on:change={onChange}
		class="h-2 accent-red-500 w-full cursor-pointer rounded-full bg-gray-200 disabled:cursor-not-allowed appearance-none"
	/>
</div>

<style>
	input[type='range']::-webkit-slider-thumb,
	input[type='range']::-moz-range-thumb {
		-webkit-appearance: none;
		appearance: none;
		width: 5px;
		height: 5px;
		background: red;
		cursor: pointer;
	}
</style>
