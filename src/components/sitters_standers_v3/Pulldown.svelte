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
		background: var(--color-off-purple);
		line-height:  20px;
		appearance: none; /* Ensure custom styles are applied */
		-webkit-appearance: none; /* For WebKit browsers */
		-moz-appearance: none; /* For Firefox */
		padding: 5px;
		padding-right: 30px;
		border: 1px solid #ccc;
		border-radius: 4px;
		background-image: url('data:image/svg+xml;charset=UTF-8,%3Csvg xmlns=%22http://www.w3.org/2000/svg%22 viewBox=%220 0 24 24%22 fill=%22none%22 stroke=%22currentColor%22 stroke-width=%222%22 stroke-linecap=%22round%22 stroke-linejoin=%22round%22 class=%22feather feather-chevron-down%22%3E%3Cpolyline points=%226 9 12 15 18 9%22/%3E%3C/svg%3E');
		background-repeat: no-repeat;
		height:  35px;
		text-overflow: ellipsis;
		background-position: right 10px center;
		background-size: 12px;
		font-size: 16px;
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
