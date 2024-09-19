<script>
	import { fly } from 'svelte/transition'; // Import the fly transition

	export let copy;
	export let selectedIndices;
	export let introActive;

	let shown = true;
	let flyValue = 200;

	const questionOrder = ["intro","stand_sit","other","body","environment"];
	const buttonOrder = ["Start <span>→</span>","Next <span>→</span>","Next <span>→</span>","Next <span>→</span>","Let's go <span>→</span>"]
	let questionNumber = 0;
	function toggleAnswer(index) {
		if (selectedIndices.includes(index)) {
			selectedIndices = selectedIndices.filter(i => i !== index);
		} else {
			selectedIndices = [...selectedIndices, index];
		}
	}

	function toggleShown() {
		shown = !shown;
	}

	function buttonBack() {
		questionNumber -= 1;
		flyValue = -200;
	}

	function buttonClicked() {
		flyValue = 200;
		if (questionNumber == questionOrder.length - 1) {
			introActive = false;
			shown = false;	
		}
		questionNumber += 1;
	}

	$: {
		selectedIndices, introActive;
	}
</script>

{#if introActive}
<div class="introContainer">
	{#key questionNumber}
	<div class="textContainer" in:fly={{ x: flyValue, duration: 200 }} out:fly={{ x: -flyValue, duration: 100 }}>

		<div class="questionContainer" >
			{#if introActive}
			{#if questionOrder[questionNumber] == "intro"}
			<h1>{copy.hed}</h1>
			<h2>{copy.dek}</h2>
			<h3>by <a href="https://pudding.cool/author/alvin-chang/">{copy.byline}</a></h3>
			{:else}
			{@html copy.question_text[questionOrder[questionNumber]].text}
			{/if}
			{/if}
		</div>

		<div class="answerContainer">
			{#each copy.questions as option}
			{#if option.cat == questionOrder[questionNumber]}
			<button 
			class="answerItem {selectedIndices.includes(option.index) ? 'selected' : ''}" 
			on:click={() => toggleAnswer(option.index)}>
			{@html option.short}
		</button>
		{/if}
		{/each}
	</div>
	{#if introActive}
	<div class="buttonContainer">
		{#if questionOrder[questionNumber] != "intro"}
			<button class="answerButton" on:click={buttonBack}><span>←</span> Back</button>
		{/if}
		<button class="answerButton" on:click={buttonClicked}>{@html buttonOrder[questionNumber]}</button>
	</div>
	{/if}

</div>
{/key}
</div>
{/if}

{#if shown}
	<div class="panel">
			{#each copy.questions as option}
				<button 
				class="answerItem {selectedIndices.includes(option.index) ? 'selected' : ''}" 
				on:click={() => toggleAnswer(option.index)}>
				{@html option.short}
				</button>
		{/each}
	</div>
{/if}


{#if !introActive}
<button class="hideShow" on:click={toggleShown}>
	{#if shown}
	x Close job requirements
	{:else}
	✎ Edit job requirements
	{/if}
</button>
{/if}

<style>
	
	h1 {
		font-weight: bold;
		font-size: 18px;
	}
	h2 {
		font-size: 15px;
	}
	h3 {
		font-size: 15px;
	}
	h3 a {
		color: var(--color-lighter-purple);
		text-decoration: none;
		border-bottom: 1px solid var(--color-lighter-purple);
	}
	h3 a:hover {
		border-bottom: 1px solid var(--color-light-purple);
	}
	.introContainer {
		background: #211d21;
		position: fixed;
		left: 0;
		top: 0;
		width: 100%;
		height: 100%;
		z-index: 10000;
	}
	.panel {
		background: none;
		position: fixed;
		left: auto;
		right: 20px;
		border: 1px solid var(--color-light-purple);
		top: 55px;
		width: 320px;
		height: 50vh;
		padding: 0px;
		overflow-y: scroll;
		scrollbar-width: thin;
		scrollbar-color: var(--color-light-purple) var(--color-dark-purple);
	}
	.textContainer {
		max-width: 320px;
		width: 100%;
		text-align: center;
		position: absolute;
		left: 50%;
		top: 40%;
		transform: translate(-50%, -50%);
	}
	.introInactive .textContainer {
		position: relative;
		width: 100%;
		left: auto;
		right: 0;
		top: 0px;
		transform: translate(0%, 0%);
	}
	.questionContainer {
		margin-bottom: 13px;
	}
	.answerContainer {
		max-height: 60vh;
	}
	.buttonContainer {
		margin-top: 20px;
	}

	.hideShow {
		position: fixed;
		right: 20px;
		top: 20px;
		background: var(--color-dark-purple);
		border: 1px solid #473847;
		cursor: pointer;
		font-size: 14px;
		z-index: 10001;
	}
	.hideShow:hover {
		border: 1px solid #755475;
	}
</style>