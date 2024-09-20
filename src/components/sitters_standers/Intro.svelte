<script>
	import { fly, fade } from 'svelte/transition'; // Import the fly transition
	import { onMount, onDestroy } from 'svelte';
	export let copy;
	export let selectedIndices;
	export let introActive;

	let shown = true;
	let flyValue = 200;

	const questionOrder = ["intro","stand_sit","body","other","environment"];
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
		if (questionNumber > 0) {
			questionNumber -= 1;
			flyValue = -200;
		}
	}

	function buttonClicked() {
		flyValue = 200;
		if (questionNumber == questionOrder.length - 1) {
			introActive = false;
			shown = false;	
		}
		questionNumber += 1;
	}

	// Handle key events for navigation
	function handleKeydown(event) {
		if (event.key === 'ArrowRight') {
			buttonClicked();
		} else if (event.key === 'ArrowLeft') {
			buttonBack();
		}
	}

	// Add and remove event listeners when component mounts and unmounts
	onMount(() => {
		if (typeof window !== 'undefined') {
			window.addEventListener('keydown', handleKeydown);
		}
	});

	// Cleanup event listener
	onDestroy(() => {
		if (typeof window !== 'undefined') {
			window.removeEventListener('keydown', handleKeydown);
		}
	});
	$: {
		selectedIndices, introActive;
	}
</script>

{#if introActive}
<div class="introContainer">
	
	<div class="textContainer">
		

		{#key questionNumber}
		<div class="slideContainer" in:fly={{ x: flyValue, duration: 300 }} out:fly={{ x: -flyValue, duration: 300 }}>
			<div class="questionContainer">
				{#if introActive}
				{#if questionOrder[questionNumber] == "intro"}
				<h1>{copy.hed}</h1>
				{:else}
				{@html copy.question_text[questionOrder[questionNumber]].text}
				{/if}
				{/if}
			</div>

			<div class="answerContainer">
				{#if questionOrder[questionNumber] == "intro"}
				<h2>{copy.dek}</h2>
				<h3>by <a href="https://pudding.cool/author/alvin-chang/">{copy.byline}</a></h3>
				{:else}
				{#each copy.questions as option}
				{#if option.cat == questionOrder[questionNumber]}
				<button  class="answerItem {selectedIndices.includes(option.index) ? 'selected' : ''}"  on:click={() => toggleAnswer(option.index)}>
					{@html option.short}
				</button>
				{/if}
				{/each}
				{/if}
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
		{#if questionNumber > 0}
		<div class="dotContainer" transition:fade>
			{#each questionOrder as q, i}
			{#if i != 0}
			<span class="dot {questionNumber >= i ? 'selected' : ''}"></span>
			{/if}
			{/each}
		</div>
		{/if}

	</div>


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
		background: var(--color-bg);
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
		height: 420px;
		text-align: center;
		position: relative;
		display: block;
		left: 50%;
		top: 50%;
		transform: translate(-50%, -50%);
	}
	.textContainer .slideContainer {
		position: absolute;
		left: 50%;
		width: 100%;
		top: 50%;
		transform: translate(-50%, -50%);
	}
	.dotContainer {
		position: absolute;
		bottom: -40px;
		transform: translate(0%, -50%);
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
		height: 300px;
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
	@media (width <= 800px) {
		.hideShow {
			display: none;
		}
	}

	.dotContainer {
		width: 100%;
		text-align: center;
	}
	.dot {
		display: inline-block;
		width: 9px;
		height: 9px;
		margin: 0 4px;
		border-radius: 50%;
		background: var(--color-light-purple);
		opacity: 0.4;
		transition: all 200ms cubic-bezier(0.250, 0.250, 0.750, 0.750); /* linear */
		transition-timing-function: cubic-bezier(0.250, 0.250, 0.750, 0.750); /* linear */
	}
	.dot.selected {
		background: var(--color-bright-purple);
		opacity: 1;
	}
</style>