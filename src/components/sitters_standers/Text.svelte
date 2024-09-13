<script>
	import { createEventDispatcher } from 'svelte';
	import Pulldown from "$components/sitters_standers/Pulldown.svelte"; 

	export let currentStageNumber, copy, data, currentVar;

	const dispatch = createEventDispatcher();

	// This function now dispatches both the answer and the currentVar to the parent
	function button(value) {
		dispatch('updateQuestion', { answer: value, currentVar });
	}

	// When the Pulldown component changes, we update currentVar and dispatch it
	function handlePulldownChange(event) {
		currentVar = event.detail.currentVar;
		dispatch('updateCurrentVar', { currentVar }); // Dispatch the updated value to the parent
	}

	function getPercentKeys(arr) {
		if (arr.length === 0) {
			return [];
		}
		const firstObject = arr[0];
		let percentKeys = Object.keys(firstObject).filter(key => key.startsWith("Percent"));
		return percentKeys;
	}

	$: {		
		currentVar;
	}
</script>

<div class="question">
	{@html copy.story[currentStageNumber].text}
	{#if copy.story[currentStageNumber].stage == "explore"}
		<Pulldown data={getPercentKeys(data)} {currentVar} on:change={handlePulldownChange} />
	{/if}
	<div class="answers stage_{currentStageNumber}">
		<button class="answerButton back" on:click={() => button(-1)}><span>←</span> Back</button>
		{#if currentStageNumber < copy.story.length - 1}
		<button class="answerButton" on:click={() => button(1)}>Next <span>→</span></button>
		{/if}
	</div>	
</div>

<style>

</style>