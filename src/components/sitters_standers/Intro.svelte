<script>
	import { fly, fade } from 'svelte/transition'; // Import the fly transition
	import { onMount, onDestroy } from 'svelte';
	import Fuzzy from "svelte-fuzzy";

	export let copy, data, currentStageNumber;
	export let selectedIndices;
	export let introActive;
	export let searchValue;
	export let selectedSalary;
	export let selectedStandingPct;
	let searchIndex = -999;

	let shown = false;
	let jobSearchShown = false;
	let flyValue = 200;
	let query = searchValue;
	let prev_query;
	let formatted = [];
	let options = { keys: ["OCCUPATION"] };
	let selectedSalaryIndex = null;
	let selectedStandingIndex = null;
	let nodatashown = false;
	let isNextDisabled = false;
	let isInputFocused = false;
	let searchShown = false;

	const questionOrder = ["intro","stand_sit","body","other","salary"];
	const buttonOrder = ["Start <span>→</span>","Next <span>→</span>","Next <span>→</span>","Next <span>→</span>","Next <span>→</span>","Let's go <span>→</span>"]
	export let questionNumber;
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
		if (jobSearchShown) {
			shown = false;
		}
	}

	function getMidpoint(n) {
        // Handle the "$150,000+" case separately
		if (n === "$150,000+") {
            return 150000; // Or any higher value you want to represent
        }

        // Split the range into numbers
        const [low, high] = n
        .replace(/\$|,|%/g, '') // Remove "$" and commas
        .split(' to ')        // Split by " to "
        .map(Number);         // Convert to numbers
        // Calculate the midpoint
        return (low + high) / 2;
    }

    function addSalary(salary, index) {
    	if (selectedSalaryIndex !== index) {
	        selectedSalaryIndex = index; // Set the new selected index
	        selectedSalary = salary === "Prefer not to answer" ? -1 : getMidpoint(salary);
	    }
	}

	function addStanding(hours, index) {
	    // Only set the selection if it's not currently selected
		if (selectedStandingIndex !== index) {
	        selectedStandingIndex = index; // Set the new selected index
	        selectedStandingPct = hours === "I don't work" ? -1 : getMidpoint(hours);
	    }
	}

	function skipQuestions() {
		selectedStandingPct = -1;
		selectedSalary = -1;
		introActive = false;
		shown = false;	
	}


	function buttonBack() {
		if (questionNumber > 0) {
			questionNumber -= 1;
			flyValue = -200;
		}
	}

	function searchJob(j) {
		nodatashown = false;
	    searchValue = j.map(obj => obj.text).join(''); // Update searchValue from selected item
	    query = searchValue; // Sync input box with searchValue
	    searchIndex = data.findIndex(obj => obj.OCCUPATION === searchValue);
	    nodatashown = data[searchIndex]?.score === -1 ? true : false;
	    searchShown = false;
	}


	
	function inputFocus() {
		isInputFocused = true;
	    query = "";        // Clear the input box
	    searchValue = "";  // Clear searchValue
	    searchShown = true;
	}

	// Function to handle input focus
	function handleInputFocus() {
		isInputFocused = true;
	}

	function handleInputBlur() {
		isInputFocused = false;
	    searchValue = query; // Sync searchValue with query on blur
	}

	function buttonClicked() {
		if (selectedStandingPct == -1 && questionNumber == 1) {
			skipQuestions();
		} 
		flyValue = 200;
		if (questionNumber == questionOrder.length - 1) {
			introActive = false;
			shown = false;	
		}
		questionNumber += 1;
	}

	function handleKeydown(event) {
	    // Check if the "Next" action should be disabled based on question number and selection status
		if ((questionNumber === 1 && selectedStandingPct === undefined) || (questionNumber === 4 && selectedSalary === undefined))  {
			isNextDisabled = true;
		} 

		if (isNextDisabled && event.key === 'ArrowRight') {
	        return; // Prevent advancing if the "Next" action should be disabled
	    }

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
		if (searchValue !== query && !isInputFocused) {
			query = searchValue;
		}
		isNextDisabled = 
		(questionNumber === 1 && selectedStandingPct === undefined) || 
		(questionNumber === 4 && selectedSalary === undefined);
		selectedIndices, introActive, searchValue, formatted, selectedSalary, selectedSalaryIndex;
		if (searchValue == "") {
			nodatashown = false;
		}
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
				<!-- <h2>{copy.dek}</h2> -->
				<h3>by <a href="https://pudding.cool/author/alvin-chang/">{copy.byline}</a></h3>
				<div class="introImage">
					<img src="assets/app/Lead.gif" alt="animation of left person washing clothes, right person typing" />
				</div>
				{:else if questionOrder[questionNumber] == "salary"}
				{#each ["$0 to $30,000","$30,000 to $60,000","$60,000 to $90,000","$90,000 to $120,000","$120,000 to $150,000","$150,000+","Prefer not to answer"] as salary, index}
				<button class="answerItem {selectedSalaryIndex === index ? 'selected' : ''}" on:click={() => addSalary(salary, index)}>
					{@html salary}
				</button>
				{/each}
				{:else if questionOrder[questionNumber] == "stand_sit"}
				{#each ["0 to 20%","20 to 40%","40 to 60%","60 to 80%","80 to 100%","I don't work"] as hours, index}
				<button class="answerItem {selectedStandingIndex === index ? 'selected' : ''}" on:click={() => addStanding(hours, index)}>
					{@html hours}
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
				{#if questionOrder[questionNumber] != "intro"}
				<button 
				class="answerButton" 
				on:click={buttonClicked} 
				disabled={isNextDisabled}>
				{@html buttonOrder[questionNumber]}
			</button>
			{:else}
			<button class="answerButton" on:click={buttonClicked}>{@html buttonOrder[questionNumber]}</button>
			{/if}
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



{#if !introActive && copy.story[currentStageNumber].stage == "explore"}
<div class="panel jobSearch">
	<input 
	bind:value={query} 
	on:focus={inputFocus} 
	on:blur={handleInputBlur} 
	placeholder="Search job titles..."
	/>
	<Fuzzy {query} {data} {options} bind:formatted />

	{#if searchShown}
	{#each formatted.slice(0, 4) as item}
	{#each item as line}
	<button class="answerItem" on:click={() => searchJob(line)}>
		{#each line as { matches, text }}
		{text}
		{/each}
	</button>
	{/each}
	{/each}
	{/if}
	{#if nodatashown}
	<div class="nodata">
		&#9888; This job doesn't have data for this variable. Choose another job or variable.
	</div>
	{/if}
</div>
{/if}

{#if introActive}
{#if questionOrder[questionNumber] == "intro"}
<div class="videoTeaser"><a href="https://www.youtube.com/watch?v=sE_Ew0Be4qE">Want a video story instead? Click here.</a></div>
{/if}
{/if}
<style>
	
	h1 {
		font-weight: bold;
		font-size: 16px;
		width: 300px;
		margin: 0 auto;
		line-height: 20px;
	}
	h2 {
		font-size: 14px;
		line-height: 18px;
	}
	h3 {
		font-size: 14px;
		line-height: 18px;
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
		right: 27px;
		border: 1px solid var(--color-light-purple);
		top: 210px;
		width: 320px;
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
		font-size:  16px;
		background: var(--color-off-purple);
	}

	.jobSearch input:focus {
		outline: 0;
		border: none;
	}
	.textContainer {
		max-width: 95%;
		width: 380px;
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
		position: fixed;
		border: 1px solid #473847;
		cursor: pointer;
		z-index: 10001;
		width: 140px;
		text-align: center;
		user-select: none;
	}
	.toolbutton.selected {
		background: var(--color-lighter-purple);
		border: 1px solid var(--color-light-purple);
	}
	.toolbutton.hideShow, .toolbutton.jobSearch {
		left: 20px;
		top: 220px;
	}
	.toolbutton:hover {
		z-index: 99999;
	}
	@media (width <= 800px) {
		.toolbutton {
			display: none;
		}
		.toolbutton.hideShow {
			left:  auto;
			right: 20px;
			top: 20px;
		}
		.toolbutton.jobSearch {
			left:  auto;
			right: 20px;
			top: 20px;
		}
		.panel {
			right:  auto;
			left: 70px;
			width:  calc(100% - 80px);
			top: 20px;
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
	.nodata {
		position: absolute;
		right: 10px;
		top: calc(100% + 10px);
		font-size: 14px;
		color: rgb(252, 186, 3);
	}
	.introImage {
		width: 320px;
		height: 200px;
		display: block;
		font-size: 9px;
		position: relative;
		pointer-events: none;
}
.introImage img {
	position: absolute;
	top:  -50px;
	left:  5%;
	width:  95%;
}

.videoTeaser {
	position: fixed;
	color: white;
	bottom: 20px;
	left: 50%;
	transform: translateX(-50%);
	text-align: center;
	padding: 20px;
	max-width: 500px;
	width: 90%;
	z-index: 99999;
}
.videoTeaser a {
	text-decoration: none !important;
	background:  var(--color-lessdark-purple);
	padding:  9px;
	border-radius:  4px;
	color:  var(--color-lighter-purple) !important;
}
.videoTeaser a:hover {
	background:  #522f52;
}
</style>