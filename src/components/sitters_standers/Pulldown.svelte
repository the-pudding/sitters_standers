<script>
	import { createEventDispatcher } from 'svelte';

	export let data;
	export let currentVar;

	const dispatch = createEventDispatcher();

	// Watch for changes in currentVar and dispatch the change event
	$:  dispatch('change', { currentVar });
</script>

<select id="pulldown" bind:value={currentVar}>
	{#each data as item}
		<option value={item}>{item}</option>
	{/each}
</select>

<style>
	#pulldown {
		left: 10px;
		top: 30%;
		width: 300px;
		z-index: 99999;
	}
</style>
