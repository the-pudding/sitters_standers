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
	            selectedIndices.includes(Number(question.index)) ? Number(question.threshold) : -1
	        ]);

	    // Step 1: Collect all the scores
	    let scoresWithIndices = [];

	    for (let i = 0; i < data.length; i++) {
	        data[i]["score"] = 0;

	        // Step 2: Calculate the score for each data point
	        for (let j = 0; j < selectedVariables.length; j++) {
	            const varname = selectedVariables[j][0];
	            const threshold = selectedVariables[j][1];
	            const value = Number(data[i][varname].toString().replace(/[^0-9.]/g, ''));

	            if (threshold == -1) {
	                data[i].score -= value;
	            } else {
	                data[i].score += value;
	            }
	        }

	        // Add the score along with its index to scoresWithIndices array
	        scoresWithIndices.push({ index: i, score: data[i].score });
	    }

	    // Step 3: Sort the scoresWithIndices array by score in ascending order
	    scoresWithIndices.sort((a, b) => a.score - b.score);

	    const numberOfIntroJobs = 10;
	    // Step 4: Get the bottom 5 scores (first 5 in the sorted array)
	    minIndicies = scoresWithIndices.slice(0, numberOfIntroJobs).map(item => item.index);

	    // Step 5: Get the top 5 scores (last 5 in the sorted array)
	    maxIndicies = scoresWithIndices.slice(-numberOfIntroJobs).map(item => item.index);

	    const smoothingAmount = 12;

	    // Step 7: Calculate the average of the lowest 10 scores (optional, as per your original code)
	    let sortedScores = scoresWithIndices.map(item => item.score);
	    let minAvg = sortedScores.slice(0, smoothingAmount).reduce((sum, score) => sum + score, 0) / smoothingAmount;

	    // Step 8: Calculate the average of the highest 10 scores (optional, as per your original code)
	    let maxAvg = sortedScores.slice(-smoothingAmount).reduce((sum, score) => sum + score, 0) / smoothingAmount;

	    // Step 9: Set minmax to these average values (optional, as per your original code)
	    minmax[0] = minAvg;
	    minmax[1] = maxAvg;
	}

	$: {
		if (currentIntroActive != introActive) {
			updateData();
			currentIntroActive = introActive;
		}
		currentVar, currentQuestionNum, minmax, minIndicies, maxIndicies;
	}
</script>

<div class="container">
	{#if introActive}
	<div transition:fade>
		<Intro {copy} bind:selectedIndices bind:introActive/>
	</div>
	{/if}
	{#if !introActive}
	<div class="canvasContainer" transition:fade>
		<Canvas {selectedIndices} {data} {copy} questions={currentQuestionOrder} {currentVar} {currentData} {currentQuestionNum} {currentStageNumber} {minmax} {minIndicies} {maxIndicies}/>
		<Axis {currentStageNumber} />
		<Text {currentStageNumber} {copy} {data} {currentVar} on:updateCurrentVar={handleCurrentVarUpdate} on:updateQuestion={handleUpdateQuestion}  />
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
		background: #211d21;
	}
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
