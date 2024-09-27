<script>
	import Canvas from "$components/sitters_standers/Canvas.svelte"; 
	import { fade } from 'svelte/transition';
	import Pulldown from "$components/sitters_standers/Pulldown.svelte"; 
	import Text from "$components/sitters_standers/Text.svelte"; 
	import Axis from "$components/sitters_standers/Axis.svelte"; 
	import Intro from "$components/sitters_standers/Intro.svelte"; 
	export let copy;
	export let data;
	export let currentQuestionNum = 0;
	let currentStageNumber = 0;
	let selectedIndices = [];
	let prevSelectedIndices = [];
	let searchValue = "";

	/// CUSTOMIZED USER CHECKPOINT

	/*------ set to true when not testing
	-----------------------------------------------*/
	let introActive = true;


	let currentIntroActive = true;
	let minmax = [-1,1];
	let minIndicies = [];
	let maxIndicies = []

	let currentData = Array(copy.questions.length).fill(1);
	let currentVar = "";
	let currentQuestionOrder = copy.questions.map(question => question.variable);

	data = data.sort((a, b) => b.TOT_EMP - a.TOT_EMP);

	function handleUpdateQuestion(event, num) {
		currentStageNumber = currentStageNumber + event.detail.answer; // answer is 1 for forward, -1 for backward 
		currentVar = copy.story[currentStageNumber].hl;
	}

	function handleCurrentVarUpdate(event) {
		currentVar = event.detail.currentVar;
	}

	function updateData() {
		let selectedVariables = copy.questions
		.filter(question => question.variable.startsWith("Percent"))
		.map(question => [
			question.variable,
			selectedIndices.includes(Number(question.index)) ? 1 : -1,
			question.cat
			]);

	    // Step 1: Collect all the scores
		let scoresWithIndices = [];

		for (let i = 0; i < data.length; i++) {
			data[i]["score"] = 0;

	        // Step 2: Calculate the score for each data point
			for (let j = 0; j < selectedVariables.length; j++) {
				const varname = selectedVariables[j][0];
				const qanswer = selectedVariables[j][1];
				const varcat = selectedVariables[j][2];
				const value = Number(data[i][varname].toString().replace(/[^0-9.]/g, ''));
				if (varcat != "noquestion") {
					if (qanswer == -1) {
						data[i].score -= value;
					} else {
						data[i].score += value;
					}
				}
			}

	        // Add the score along with its index to scoresWithIndices array
			scoresWithIndices.push({ index: i, score: data[i].score, dots: data[i].dots });
		}

	    // Step 3: Sort the scoresWithIndices array by score in ascending order
		scoresWithIndices.sort((a, b) => a.score - b.score);

		const numberOfIntroJobs = 10;
	    // Step 4: Get the bottom 10 scores (first 10 in the sorted array)
		minIndicies = scoresWithIndices.slice(0, numberOfIntroJobs).map(item => item.index);

	    // Step 5: Get the top 10 scores (last 10 in the sorted array)
		maxIndicies = scoresWithIndices.slice(-numberOfIntroJobs).map(item => item.index);

		const smoothingAmount = 20;

	    // Step 7: Calculate the average of the lowest 10 scores (optional, as per your original code)
		let sortedScores = scoresWithIndices.map(item => item.score);
		let minAvg = sortedScores.slice(0, smoothingAmount).reduce((sum, score) => sum + score, 0) / smoothingAmount;

	    // Step 8: Calculate the average of the highest 10 scores (optional, as per your original code)
		let maxAvg = sortedScores.slice(-smoothingAmount).reduce((sum, score) => sum + score, 0) / smoothingAmount;

	    // Step 9: Set minmax to these average values (optional, as per your original code)
		minmax[0] = minAvg;
		minmax[1] = maxAvg;

	    // Step 10: Move the first item with at least 4 dots to the front of maxIndicies
		let found = false;

	    // Check maxIndicies for an item with at least 4 dots
		for (let i = maxIndicies.length - 1; i >= 0; i--) {
			const index = maxIndicies[i];
			if (data[index].dots >= 6) {
	            // Remove the found item from its current position
				maxIndicies.splice(i, 1);
	            // Insert it at the front of the array
				maxIndicies.unshift(index);
				found = true;
				break;
			}
		}

	    // If no item in maxIndicies has at least 4 dots, find one in the entire sorted array
		if (!found) {
			for (let i = scoresWithIndices.length - 1; i >= 0; i--) {
				if (scoresWithIndices[i].dots >= 4) {
					const index = scoresWithIndices[i].index;
	                // Insert it at the front of maxIndicies
					maxIndicies.unshift(index);
	                // Remove the last item to maintain the number of intro jobs
					if (maxIndicies.length > numberOfIntroJobs) {
						maxIndicies.pop();
					}
					break;
				}
			}
		}

	    // console.log(data);
	}

	$: {
		if (currentIntroActive != introActive || selectedIndices != prevSelectedIndices) {
			updateData();
			currentIntroActive = introActive;
			prevSelectedIndices = selectedIndices; 
		}
		currentVar, currentQuestionNum, minmax, minIndicies, maxIndicies, searchValue;
	}
</script>

<div class="container">
	<Intro {copy} {data} bind:searchValue bind:selectedIndices bind:introActive/>
	{#if !introActive}
	<div class="canvasContainer" transition:fade>
		<Canvas {searchValue} {selectedIndices} {data} {copy} questions={currentQuestionOrder} {currentVar} {currentData} {currentQuestionNum} {currentStageNumber} {minmax} {minIndicies} {maxIndicies}/>
		<Axis {currentStageNumber} />
		<Text bind:searchValue {currentStageNumber} {copy} {data} {currentVar} on:updateCurrentVar={handleCurrentVarUpdate} on:updateQuestion={handleUpdateQuestion}  />
	</div>
	{/if}
</div>

<style>
	.container {
		position: fixed;
		width: 100%;
		left: 0px;
		top: 0px;
		height: 100vh;
		background: var(--color-bg);
	}
/*	@media (width <= 1000px) {*/
		.container {
			height: calc(100% - 150px);
			bottom: 150px;
		}
/*	}*/
	.canvasContainer {
		cursor: move; /* fallback if grab cursor is unsupported */
		cursor: grab;
		cursor: -moz-grab;
		cursor: -webkit-grab;
	}
	.canvasContainer:active {
		cursor: grabbing;
		cursor: -moz-grabbing;
		cursor: -webkit-grabbing;
	}
</style>
