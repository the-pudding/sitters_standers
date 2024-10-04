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
	let selectedIndices = [];
	let prevSelectedIndices = [];
	let searchValue = "";
	let selectedSalary = 50000;
	let explored = false;

	/*------ set to true when not testing
	-----------------------------------------------*/
	let introActive = true;


	let currentIntroActive = true;
	let minmax = [-1,1];
	let minIndicies = [];
	let maxIndicies = []

	let currentData = Array(copy.questions.length).fill(1);
	let currentVar = undefined;
	let prevVar = undefined;
	let currentQuestionOrder = copy.questions.map(question => question.variable);
	let axis_variable = "";

	data = data.sort((a, b) => b.TOT_EMP - a.TOT_EMP);
	let you = {"OCCUPATION": "You", "OCC_SHORT": "You", "A_MEAN": selectedSalary};
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
		}
		data[data.length-1]["A_MEAN"] = selectedSalary;

	    // Step 1: Collect all the scores
		let scoresWithIndices = [];

		for (let i = 0; i < data.length; i++) {
			data[i]["score"] = 0;
			data[i]["similarity_score"] = 0;
			
			// for similarity score
			for (let j = 0; j < selectedVariables.length; j++) {
				const varname = selectedVariables[j][0];
				const qanswer = selectedVariables[j][1];
				const varcat = selectedVariables[j][2];
				let sim_value = 0;
				if (varname in data[i]) {
					sim_value = Number(data[i][varname].toString().replace(/[^0-9.]/g, ''));
				}
				if (varcat != "noquestion") {
					if (qanswer == 0) {
						data[i]["similarity_score"] -= sim_value;
					} else {
						data[i]["similarity_score"] += sim_value;
					}
				}
			}
			if (copy.story[currentStageNumber].hl == "" || copy.story[currentStageNumber].hl == undefined) {
				data[i].score = data[i]["similarity_score"];
			} else {
				// for variable score
				let value = 0;
				if (currentVar in data[i]) {
					if (data[i][currentVar] == "") {
						value = -1;
					} else {
						value = Number(data[i][currentVar].toString().replace(/[^0-9.]/g, ''));
					}
				}
				data[i].score += value;
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
		// maxIndicies.push(data.length-1)
		const smoothingAmount = 5;

	    // Step 7: Calculate the average of the lowest smoothingAmount scores
		let sortedScores = scoresWithIndices.map(item => item.score);

		let minAvg = sortedScores.slice(0, smoothingAmount).reduce((sum, score) => sum + score, 0) / smoothingAmount;

	    // Step 8: Calculate the average of the highest smoothingAmount scores
		let maxAvg = sortedScores.slice(-smoothingAmount).reduce((sum, score) => sum + score, 0) / smoothingAmount;

	    // Step 9: Set minmax to these average values
		minmax[0] = minAvg;
		minmax[1] = maxAvg;

	    // Step 10: Move the first item with at least 6 dots to the front of maxIndicies
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
				if (scoresWithIndices[i].dots >= 6) {
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
	}

	$: {
		if (currentIntroActive != introActive || selectedIndices != prevSelectedIndices || currentVar != prevVar) {
			updateData();
			currentIntroActive = introActive;
			prevSelectedIndices = selectedIndices; 
			prevVar = currentVar;
		}
		let foundObject = copy.questions.find(obj => obj.variable === copy.story[currentStageNumber].hl);
		axis_variable = foundObject ? foundObject.axis_variable : "like your job"; // Safely access axis_variable
		currentVar, currentQuestionNum, minmax, minIndicies, maxIndicies, searchValue, selectedSalary;
		updateData();
	}
</script>

<div class="container">
	<Intro {copy} {data} {currentStageNumber} bind:selectedSalary bind:searchValue bind:selectedIndices bind:introActive/>
	{#if copy.story[currentStageNumber].cat == "ExploreInstruction" && !explored}
	<div class="exploreInstruction" transition:fade>
		<img src="assets/app/hand.png"/>
		<!-- <div class="exploreWord">Explore</div> -->
	</div>
	{/if}
	{#if !introActive}
	<div class="canvasContainer" transition:fade>
		<Canvas {searchValue} {selectedIndices} {data} {copy} questions={currentQuestionOrder} {currentVar} {currentData} {currentQuestionNum} {currentStageNumber} {minmax} {minIndicies} {maxIndicies} bind:explored/>
		<Axis {currentStageNumber} {axis_variable} />
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

.exploreInstruction {
/*	background: rgba(0,0,0,.8);*/
	border-radius:10px;
	color: black;
	position: fixed;
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
	width: 100px;
	height: 100px;
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
