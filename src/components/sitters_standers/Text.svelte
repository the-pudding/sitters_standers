<script>
	import { createEventDispatcher } from 'svelte';
	import { onMount, onDestroy } from 'svelte';
	import Pulldown from "$components/sitters_standers/Pulldown.svelte"; 

	export let currentStageNumber, copy, data, currentVar, searchValue;

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

	function convertToHTML(text) {
		let finalText = [];
		if (text != undefined) {
			let textArray = text.split(/(\n\n|\r\r|\n\r|\r\n)/);
			textArray.forEach(function(line) {
				if (line.indexOf("Component|") != -1) {
					let compName = line.split("|")[1];
					line = `<svelte:component this=${compName}></svelte:component>`
				}
				
				if (line.indexOf("IMAGE|") != -1) {
					line = '<div class="imageContainer"><img class="desktopImage" src="assets/leftovers/' + line.replace("IMAGE|","").replace(/(\r\n|\n|\r)/gm, "") + '.svg"/><img class="mobileImage" src="assets/leftovers/' + line.replace("IMAGE|","").replace(/(\r\n|\n|\r)/gm, "") + '_mobile.svg"/></div>';
				}
				if (line.indexOf(">>") != -1) {
					line = "<div class='chartPlaceholder'>" + line + "</div>";
				}
				if (/[A-Za-z0-9]/.test(line)) {
					finalText.push(line);	
				}				
			})
			return wrapInPTags(finalText);
		}
	}

	function wrapInPTags(arr) {
		return arr.map(item => {
			if (/^<[^>]+>.*<\/[^>]+>$/.test(item)) {
				return item;
			} else {
				return `<p>${item}</p>`;
			}
		}).join('');
	}

	// Handle key events for navigation
	function handleKeydown(event) {
		if (event.key === 'ArrowRight') {
			if (currentStageNumber < copy.story.length -1) {
				button(1);
			}
		} else if (event.key === 'ArrowLeft') {
			if (currentStageNumber > 0) {
				button(-1);
			}
		}
	}

	function skip() {
		if (currentStageNumber == copy.story.length - 1) {
			button(-currentStageNumber);
		} else {
			button(copy.story.length - currentStageNumber - 1);	
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
		currentVar;
		if (copy.story[currentStageNumber].job != undefined) {
			searchValue = copy.story[currentStageNumber].job;
		} else {
			searchValue = "";
		}
	}
</script>

<div class="question">
	<div class="progressBar" style="width: {currentStageNumber/(copy.story.length - 1)*100}%"></div>
	<div class="kicker">{copy.story[currentStageNumber].cat}</div>
	<div class="text">
		{@html convertToHTML(copy.story[currentStageNumber].text)}
	</div>
	
	<div class="answers stage_{currentStageNumber}">
		{#if currentStageNumber < copy.story.length - 1}
			<button class="answerButton back" on:click={() => button(-1)}><span>←</span> Back</button>
			<button class="answerButton" on:click={() => button(1)}>Next <span>→</span></button>
		{/if}
		{#if copy.story[currentStageNumber].stage == "explore"}
		<Pulldown opts={copy.questions} {currentVar} on:change={handlePulldownChange} />
		{/if}
	</div>
	<button class="skipToExplore" on:click={skip}>
		{#if currentStageNumber < copy.story.length - 1}
		Skip to explore
		{:else}
		Reset to start
		{/if}
	</button>
</div>

<style>
.skipToExplore {
	background: none;
	color: white;
	text-align: left;
	padding: 0;
	margin-top: 10px;
	font-size: 13px;
	color: var(--color-light-purple);
	position: absolute;
	bottom: 15px;
	left: 10px;
}
.skipToExplore:hover {
	color: var(--color-off-purple);
	text-decoration: underline;
}
</style>