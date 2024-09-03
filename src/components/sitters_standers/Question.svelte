<script>
	import { createEventDispatcher } from 'svelte';

	export let currentQuestionNum;
	export let copy;

	const dispatch = createEventDispatcher();

	function nextQuestion(value) {
		dispatch('updateQuestion', { direction: 1, answer: value });
	}

	function previousQuestion() {
		dispatch('updateQuestion', { direction: -1 });
	}
</script>

<div class="question">
	<div class="back">
		<button class="backButton" on:click={previousQuestion}>Back</button>
	</div>
	{#if copy.questions[currentQuestionNum].variable != "end"}
	{@html copy.questions[currentQuestionNum].num + ". " + copy.questions[currentQuestionNum].text}
	<div class="answers">
		<button class="answerButton" on:click={() => nextQuestion(0)}><span>←</span> No</button>
		<button class="answerButton" on:click={() => nextQuestion(1)}>Yes <span>→</span></button>
	</div>
	{:else}
		End text TKTKTKTKT.
	{/if}
	
</div>

<style>

</style>