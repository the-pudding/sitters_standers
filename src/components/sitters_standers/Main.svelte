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
	export let currentStageNumber;
	export let x_axis_variable;
	let questionNumber = 0;
	let selectedIndices = [];
	let prevSelectedIndices = [];
	let searchValue = "";
	let selectedSalary = 50000;
	let selectedStandingPct = 20;
	let explored = false;
	let bg;
	let reset = false;
	
	let x_axis_variable_range = {
		"A_MEAN": [30000,110000, "Less money","More money"],
		"nonwhite_pct": [0,.5, "Fewer nonwhite people","More nonwhite people"],
		"noncitizen_pct": [0,0.2,"Fewer noncitizens","More noncitizens"],
		"Percent of day where standing is required, mean": [0,100,"Sitters","Standers"],
		"PCT_STAND": [0,100,"Sitters","Standers"]
	}



	/*------ set to true when not testing
	-----------------------------------------------*/
	export let introActive;


	let currentIntroActive = true;
	let minmax = [-1,1];
	let minIndicies = [];
	let maxIndicies = [];

	let currentData = Array(copy.questions.length).fill(1);
	let currentVar = undefined;
	let prevVar = undefined;
	let currentQuestionOrder = copy.questions.map(question => question.variable);
	let axis_variable = "";
	let axis_flip = "";

	data = data.sort((a, b) => b.TOT_EMP - a.TOT_EMP);
	let you = {"OCCUPATION": "You", "OCC_SHORT": "You", "A_MEAN": selectedSalary, "PCT_STAND": selectedStandingPct };
	data.push(you);

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
			selectedIndices.includes(Number(question.index)) ? 1 : 0,
			question.cat
			]);
		// Update You
		for (let i = 0; i < selectedVariables.length; i++) {
			const varname = selectedVariables[i][0];
			const qanswer = selectedVariables[i][1];
			const varcat = selectedVariables[i][2];
			data[data.length-1][varname] = qanswer*100;
			if (varcat == "demographics") {
				data[data.length-1][varname] = -1;
			} else if (data[data.length-1][varname] == 0) {
				data[data.length-1][varname] = 0.1;
			}
		}
		data[data.length-1]["A_MEAN"] = selectedSalary;
		data[data.length-1]["PCT_STAND"] = selectedStandingPct;
	    // Step 1: Collect all the scores
		let scoresWithIndices = [];
		let nudgeDirection = 1;
		let nudgeAmount = 0;
		for (let i = 0; i < data.length; i++) {
			data[i]["score"] = 0;
			if (copy.story[currentStageNumber].hl == undefined) {
				data[i].score = 50 + (nudgeDirection * nudgeAmount);
				nudgeDirection = nudgeDirection * -1;
				nudgeAmount += 0.08;
			} else {
				// for variable score
				let value = 0;
				if (currentVar in data[i]) {
					if (data[i][currentVar] == undefined || data[i][currentVar] == "" || data[i][x_axis_variable] == "") {
						value = -1;
					} else {
						value = Number(data[i][currentVar].toString().replace(/[^0-9.]/g, ''));
					}
				} else {
					value = -1;
				}
				data[i].score += value;
			}
	        // Add the score along with its index to scoresWithIndices array
			scoresWithIndices.push({ job: data[i].OCCUPATION, index: i, n: data[i].n, score: data[i].score, dots: data[i].dots, value: data[i][x_axis_variable] });
		}
		
	    // Sort to figure out the top X and bottom X
		const scoresWithValues = [...scoresWithIndices].sort((a, b) => a.value - b.value);
		const numberOfIntroJobs = 30;
		minIndicies = scoresWithValues
		.filter(item => item.job !== "You")
		.slice(0, numberOfIntroJobs)
		.map(item => item.index);

		maxIndicies = scoresWithValues
		.filter(item => item.job !== "You")
		.slice(-numberOfIntroJobs)
		.map(item => item.index);
		const firstMatchingItem = scoresWithIndices.find(item => item.n == 157);
		if (firstMatchingItem) {
			maxIndicies.unshift(firstMatchingItem.index);
		}
		const secondMatchingItem = scoresWithIndices.find(item => item.n == 44);
		if (secondMatchingItem) {
			minIndicies.unshift(secondMatchingItem.index);
		}

		// Get min and max
		scoresWithIndices.sort((a, b) => a.score - b.score);
		const smoothingAmounts = [3,3];
		let sortedScores = scoresWithIndices.map(item => item.score);
		let minAvg = sortedScores.slice(0, smoothingAmounts[0]).reduce((sum, score) => sum + score, 0) / smoothingAmounts[0];
		let maxAvg = sortedScores.slice(-smoothingAmounts[1]).reduce((sum, score) => sum + score, 0) / smoothingAmounts[1];
		minmax[0] = minAvg;
		minmax[1] = maxAvg;

		// Custom min-max
		if (currentVar == undefined) {
			minmax = [0,100];
		}
		if (currentVar == "white_pct") {
			minmax = [65,90];
		}
		if (currentVar == "black_pct") {
			minmax = [3,23];
		}
		if (currentVar == "hisp_pct") {
			minmax = [7,30];
		}
		if (currentVar == "asian_pct") {
			minmax = [0,18];
		}
		if (currentVar == "noncitizen_pct") {
			minmax = [2,20];
		}
		if (currentVar == "A_MEAN") {
			minmax = [30000,120000];
		}

	    // // Step 10: Move the first item with at least 6 dots to the front of maxIndicies
		// let found = false;

	    // // Check maxIndicies for an item with at least 4 dots
		// for (let i = maxIndicies.length - 1; i >= 0; i--) {
		// 	const index = maxIndicies[i];
		// 	if (data[index].dots >= 6) {
	    //         // Remove the found item from its current position
		// 		maxIndicies.splice(i, 1);
	    //         // Insert it at the front of the array
		// 		maxIndicies.unshift(index);
		// 		found = true;
		// 		break;
		// 	}
		// }

	    // // If no item in maxIndicies has at least 4 dots, find one in the entire sorted array
		// if (!found) {
		// 	for (let i = scoresWithIndices.length - 1; i >= 0; i--) {
		// 		if (scoresWithIndices[i].dots >= 6) {
		// 			const index = scoresWithIndices[i].index;
	    //             // Insert it at the front of maxIndicies
		// 			maxIndicies.unshift(index);
	    //             // Remove the last item to maintain the number of intro jobs
		// 			if (maxIndicies.length > numberOfIntroJobs) {
		// 				maxIndicies.pop();
		// 			}
		// 			break;
		// 		}
		// 	}
		// }
	}

	$: {
		if (currentIntroActive != introActive || selectedIndices != prevSelectedIndices || currentVar != prevVar) {
			explored = false;
			updateData();
			currentIntroActive = introActive;
			prevSelectedIndices = selectedIndices; 
			prevVar = currentVar;
			if (copy.story[currentStageNumber].stage != "explore") {
				searchValue = copy.story[currentStageNumber].job == undefined ? "" : copy.story[currentStageNumber].job;	
			}
		}
		
		let foundObject = copy.questions.find(obj => obj.variable === currentVar);
		axis_variable = foundObject ? foundObject.axis_variable : -1; // Safely access axis_variable
		axis_flip = foundObject ? foundObject.axis_flip : ""; // Safely access axis_flip
		bg = copy.story[currentStageNumber].bg;
		currentVar, currentQuestionNum, minmax, minIndicies, maxIndicies, searchValue, selectedSalary, selectedStandingPct;
		updateData();
	}
</script>

<div class="container">
	<Intro {copy} {data} {currentStageNumber} bind:selectedSalary bind:searchValue bind:selectedIndices bind:introActive bind:selectedStandingPct bind:questionNumber/>
	{#if copy.story[currentStageNumber].cat == "ExploreInstruction" && !explored}
	<div class="exploreInstruction" transition:fade>
		<img src="assets/app/hand.png"/>
		<!-- <div class="exploreWord">Explore</div> -->
	</div>
	{/if}
	{#if !introActive}
	<div class="canvasContainer" transition:fade>
		<Canvas {bg} {x_axis_variable} {x_axis_variable_range} {searchValue} {selectedIndices} {data} {copy} questions={currentQuestionOrder} {currentVar} {currentData} {currentQuestionNum} {currentStageNumber} {minmax} {minIndicies} {maxIndicies} bind:reset bind:explored/>
		{#key axis_variable}
		<Axis {currentStageNumber} {axis_flip} {axis_variable} {x_axis_variable} {x_axis_variable_range} />
		{/key}
		<Text bind:searchValue {currentStageNumber} {copy} {data} {currentVar} on:updateCurrentVar={handleCurrentVarUpdate} on:updateQuestion={handleUpdateQuestion} bind:introActive  bind:questionNumber bind:reset/>
	</div>
	{/if}
</div>

<style>
	.container {
		position: fixed;
		width: calc(100% - 370px);
		left: 370px;
		top: 0px;
		height: 100vh;
		background: var(--color-bg);
	}
	@media (width <= 860px) {
		.container {
			height: calc(100% - 150px);
			left:  0px;
			width:  100%;
			bottom: 150px;
		}
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

	.exploreInstruction {
/*	background: rgba(0,0,0,.8);*/
border-radius:10px;
color: black;
position: absolute;
padding: 20px;
left: 50%;
top: 50%;
width: 100px;
height: 100px;
transform: translate(-50%, -50%);
}
.exploreInstruction img {
	position: absolute;
	left: 50%;
	top: 50%;
	width: 60px;
	height: 60px;
	transform: translate(-50%, -50%);
	animation: moveRotate 4s infinite ease-in-out;
	opacity: 1;
}
.exploreWord {
	color: white;
	width: 100%;
	position: absolute;
	bottom:10px;
	left: 0px;
	text-align: center;
	font-weight: bold;
	font-family: var(--serif);
	font-size: 16px;
}
@keyframes moveRotate {
	0% {
		transform: translate(-50%, -50%) rotate(0deg) scale(1);
	}
	25% {
		transform: translate(-70%, -50%) rotate(0deg); /* Move further left */
	}
	50% {
		transform: translate(-40%, -50%) rotate(0deg); /* Move further right */
	}
	75% {
		transform: translate(-50%, -50%) rotate(20deg); /* Rotate more */
	}
	100% {
		transform: translate(-50%, -50%) rotate(0deg) scale(1); /* Back to center and no rotation */
	}
}
</style>
