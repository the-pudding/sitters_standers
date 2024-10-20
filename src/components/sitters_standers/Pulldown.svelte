<script>
	import { createEventDispatcher } from 'svelte';

	export let data, opts, currentVar;

	const dispatch = createEventDispatcher();

	// Watch for changes in currentVar and dispatch the change event
	$: {
		dispatch('change', { currentVar })
	};
</script>

<select id="pulldown" bind:value={currentVar}>
	{#each opts as opt}
		<option value={opt.variable}>{@html opt.short}</option>
	{/each}
</select>

<style>
	#pulldown {
		left: 2px;
		bottom: 0px;
		position: absolute;
		width: calc(100% - 5px);
		z-index: 99999;
	}
	option {
		height: 30px;
	}
	@media (width <= 860px) {
		#pulldown {
			left:  10px;
			width: calc(100% - 20px);
		}
	}
</style>
