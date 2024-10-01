<script>
	import { fly, fade } from 'svelte/transition'; // Import the fly transition
	import { onMount, onDestroy } from 'svelte';
	import Fuzzy from "svelte-fuzzy";

	export let copy, data;
	export let selectedIndices;
	export let introActive;
	export let searchValue;
	export let selectedSalary;

	let shown = false;
	let jobSearchShown = false;
	let flyValue = 200;
	let query;
	let prev_query;
	let formatted = [];
	let options = { keys: ["OCCUPATION"] };
	let selectedSalaryIndex = null;
	

	const questionOrder = ["intro","stand_sit","body","other","environment","salary"];
	const buttonOrder = ["Start <span>→</span>","Next <span>→</span>","Next <span>→</span>","Next <span>→</span>","Next <span>→</span>","Let's go <span>→</span>"]
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
		if (shown) {
			jobSearchShown = false;
		}
	}
	function toggleJobSearch() {
		jobSearchShown = !jobSearchShown;
		if (jobSearchShown) {
			shown = false;
		}
		if (!jobSearchShown) {
			searchValue = "";
			query = "";
		}
	}

	function getMidpoint(salary) {
        // Handle the "$150,000+" case separately
        if (salary === "$150,000+") {
            return 150000; // Or any higher value you want to represent
        }
        
        // Split the range into numbers
        const [low, high] = salary
            .replace(/\$|,/g, '') // Remove "$" and commas
            .split(' to ')        // Split by " to "
            .map(Number);         // Convert to numbers

        // Calculate the midpoint
        return (low + high) / 2;
    }
	function addSalary(salary, index) {
        if (selectedSalaryIndex === index) {
            selectedSalaryIndex = null; // Deselect if the same button is clicked again
        } else {
            selectedSalaryIndex = index; // Set the new selected index
        }
        selectedSalary = getMidpoint(salary);
    }

	function buttonBack() {
		if (questionNumber > 0) {
			questionNumber -= 1;
			flyValue = -200;
		}
	}

	function searchJob(j) {
		searchValue = j.map(obj => obj.text).join('');
		query = searchValue;
	}

	function inputFocus() {
		query = "";
		searchValue = "";
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
		selectedIndices, introActive, searchValue, formatted, selectedSalary, selectedSalaryIndex;
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
				{:else if questionOrder[questionNumber] == "salary"}
					{#each ["$0 to $30,000","$30,000 to $60,000","$60,000 to $90,000","$90,000 to $120,000","$120,000 to $150,000","$150,000+"] as salary, index}
					    <button class="answerItem {selectedSalaryIndex === index ? 'selected' : ''}" on:click={() => addSalary(salary, index)}>
					        {@html salary}
					    </button>
					{/each}
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
<div class="panel jobRequirement">
	{#each copy.questions as option}
	<button 
	class="answerItem {selectedIndices.includes(option.index) ? 'selected' : ''}" 
	on:click={() => toggleAnswer(option.index)}>
	{@html option.short}
</button>
{/each}
</div>
{/if}

{#if jobSearchShown}
<div class="panel jobSearch">
	<input bind:value={query}  on:focus={inputFocus} placeholder="Search job titles..."/>
	<Fuzzy {query} {data} {options} bind:formatted />
	{#if searchValue == ""}
	{#each formatted.slice(0, 8) as item}
	{#each item as line}
	<button class="answerItem" on:click={() => searchJob(line)}>
		{#each line as { matches, text }}
			{text}
		{/each}
	</button>
	{/each}
	{/each}
	{/if}
</div>
{/if}


{#if !introActive}
<!-- <button class="toolbutton hideShow" class:selected={shown} on:click={toggleShown}>
	{#if shown}
	x Requirements
	{:else}
	✎ Requirements
	{/if}
</button> -->
<button class="toolbutton jobSearch" class:selected={jobSearchShown} on:click={toggleJobSearch}>
	{#if !jobSearchShown && searchValue != ""}
	x Search jobs
	{:else if jobSearchShown || searchValue != ""}
	x Clear search
	{:else}
	🔍 Search jobs
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
		background: var(--color-dark-purple);;
		position: fixed;
		left: auto;
		right: 20px;
		border: 1px solid var(--color-light-purple);
		top: 53px;
		width: 279px;
		padding: 0px;
		scrollbar-width: inherit;
		scrollbar-color: var(--color-light-purple) var(--color-dark-purple);
		user-select: none;
	}
	.panel.jobRequirement {
		height: 50vh;
		max-height: 500px;
		overflow-y: scroll;
	}
	.panel.jobSearch {
		max-height: 500px;
		z-index: 9999;
	}
	.jobSearch input {
		width: 100%;
		border-radius: 0;
	}
	.jobSearch input:focus {
		outline: 0;
		border: none;
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

	.toolbutton {
		background: var(--color-dark-purple);
		color: var(--color-off-purple);
		position: fixed;
		background: var(--color-dark-purple);
		border: 1px solid #473847;
		cursor: pointer;
		font-size: 13px;
		z-index: 10001;
		width: 140px;
		text-align: center;
		border-radius: 0px;
		user-select: none;
	}
	.toolbutton.selected {
		background: var(--color-light-purple);
		border: 1px solid var(--color-light-purple);
	}
	.toolbutton.hideShow {
		right: 20px;
		top: 20px;
	}
	.toolbutton.jobSearch {
/*		right: 159px;*/
		right: 20px;
		top: 20px;
	}
	.toolbutton:hover {
		color: white;
		z-index: 99;
	}
	/*@media (width <= 800px) {
		.toolbutton {
			display: none;
		}
	}*/

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