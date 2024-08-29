<script>
	import { createEventDispatcher } from 'svelte';

	export let data;
	export let selectedVar;

	const dispatch = createEventDispatcher();

	$: {
		dispatch('change', { selectedVar });
	}
</script>

<select id="pulldown" bind:value={selectedVar}>
	{#each data as item}
		<option value={item}>{item}</option>
	{/each}
</select>

<style>
	#pulldown {
		position: fixed;
		left: 50%;
		top: 50%;
		margin-top: -20px;
		margin-left: -300px;
		width: 600px;
		z-index: 99999;
	}
</style>
