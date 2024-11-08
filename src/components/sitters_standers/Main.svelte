<script>
	import { onMount } from 'svelte';
	import Canvas from "$components/sitters_standers/Canvas.svelte"; 
	import { fade } from 'svelte/transition';
	import Pulldown from "$components/sitters_standers/Pulldown.svelte"; 
	import Text from "$components/sitters_standers/Text.svelte"; 
	import Axis from "$components/sitters_standers/Axis.svelte"; 
	import Intro from "$components/sitters_standers/Intro.svelte";
	export let copy, data, headervars;
	export let currentQuestionNum = 0;
	export let currentStageNumber;
	export let x_axis_variable;
	let questionNumber = 0;
	let selectedIndices = [];
	let prevSelectedIndices = [];
	let searchValue = "";
	let selectedSalary = undefined;
	let selectedStandingPct = undefined;
	let explored = false;
	let bg;
	let reset = false;
	let prefersReducedMotion = false;
	let showAverageStage = 9;
	
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
	let you = {"OCCUPATION": "You", "OCC_SHORT": "YOU", "A_MEAN": selectedSalary, "PCT_STAND": selectedStandingPct,"yellow": true };
	let sitter = {"OCCUPATION": "Sitter avg.", "OCC_SHORT": "AVERAGE SITTER","yellow": true };
	let stander = {"OCCUPATION": "Stander avg.", "OCC_SHORT": "AVERAGE STANDER","yellow": true };
	weightedAverage();

	function weightedAverage() {
		headervars.headers.forEach(hed => {
			sitter[hed] = -1;
			stander[hed] = -1;
			let weightedAverage = {"sitter": [0,0], "stander": [0,0]};
			data.forEach(item => {
				let val = hed in item && item[hed] && item[x_axis_variable]
				? Number(item[hed].toString().replace(/[^0-9.]/g, '')) || -1
				: -1;
				
				if (val != -1) {
					if (item[x_axis_variable] < 50) {
						weightedAverage.sitter[0] += item.TOT_EMP*val;
						weightedAverage.sitter[1] += item.TOT_EMP;
					}
					if (item[x_axis_variable] >= 50) {
						weightedAverage.stander[0] += item.TOT_EMP*val;
						weightedAverage.stander[1] += item.TOT_EMP;
					}
					
				}
			})
			sitter[hed] = weightedAverage.sitter[1] ? (weightedAverage.sitter[0] / weightedAverage.sitter[1]) : 0;
			stander[hed] = weightedAverage.stander[1] ? (weightedAverage.stander[0] / weightedAverage.stander[1]) : 0;
		})
		data.push(sitter);
		data.push(stander);
		data.push(you);
	}

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
			data[data.length - 1][varname] = qanswer * 100;
			if (varcat == "demographics") {
				data[data.length - 1][varname] = -1;
			} else if (data[data.length - 1][varname] == 0) {
				data[data.length - 1][varname] = 0.1;
			}
		}
		data[data.length - 1]["A_MEAN"] = selectedSalary;
		data[data.length - 1]["PCT_STAND"] = selectedStandingPct;

	    // Step 1: Collect all the scores
		let scoresWithIndices = [];
		let nudgeDirection = 1, nudgeAmount = 0;
		data.forEach((item, i) => {
			item.score = 0;
			if (item.OCC_SHORT === "YOU" && selectedStandingPct == -1) {
				item.score = -1;
			} else if (copy.story[currentStageNumber].hl === undefined) {
				if (item.yellow) {
					item.score = 50;
					if (item.OCC_SHORT == "YOU") {
						item.score = 55;
					}
				} else if (prefersReducedMotion) {
					item.score = 50 + (nudgeDirection * nudgeAmount);
					nudgeDirection *= -1;
					nudgeAmount += 0.05;
				} else {
					item.score = 50 + (nudgeDirection * nudgeAmount);
					nudgeDirection *= -1;
					nudgeAmount += 0.01;
				}
			} else {
				const value = currentVar in item && item[currentVar] && item[x_axis_variable]
				? Number(item[currentVar].toString().replace(/[^0-9.]/g, '')) || -1
				: -1;
				item.score += value;
			}

			scoresWithIndices.push({
				job: item.OCCUPATION,
				index: i,
				n: item.n,
				tot_emp: item.TOT_EMP,
				score: item.score,
				dots: item.dots,
				value: item[x_axis_variable]
			});
		});

	    // Sort to figure out the top X and bottom X
		const scoresWithValues = [...scoresWithIndices].sort((a, b) => a.value - b.value);
		const numberOfIntroJobs = 30;
		const filteredScores = scoresWithValues.filter(item => item.job !== "You");

		minIndicies = filteredScores.slice(0, numberOfIntroJobs).map(item => item.index);
		maxIndicies = filteredScores.slice(-numberOfIntroJobs).map(item => item.index);

		[scoresWithIndices.find(item => item.n == 157)?.index].forEach(index => {
			if (index !== undefined) { 
				maxIndicies.unshift(index)
			};
		});


		[scoresWithIndices.find(item => item.n == 44)?.index].forEach(index => {
			if (index !== undefined) {
				minIndicies.unshift(index);
			}
		});

	    // Get min and max, weighted by TOT_EMP
		scoresWithIndices.sort((a, b) => a.score - b.score);
		const smoothingAmounts = [30, 30];

	    // Calculate weighted minAvg
		let totalWeightMin = 0;
		let weightedSumMin = 0;
		for (let i = 0; i < smoothingAmounts[0]; i++) {
			const item = scoresWithIndices[i];
			if (item) {
	            const weight = item.tot_emp || 1; // Use item.TOT_EMP as the weight, default to 1
	            weightedSumMin += item.score * weight;
	            totalWeightMin += weight;
	        }
	    }
	    let minAvg = weightedSumMin / totalWeightMin;

	    // Calculate weighted maxAvg
	    let totalWeightMax = 0;
	    let weightedSumMax = 0;
	    for (let i = scoresWithIndices.length - smoothingAmounts[1]; i < scoresWithIndices.length; i++) {
	    	const item = scoresWithIndices[i];
	    	if (item) {
	            const weight = item.tot_emp || 1; // Use item.TOT_EMP as the weight, default to 1
	            weightedSumMax += item.score * weight;
	            totalWeightMax += weight;
	        }
	    }
	    let maxAvg = weightedSumMax / totalWeightMax;

	    minmax[0] = minAvg;
	    minmax[1] = maxAvg;

	    const minmaxMap = {
	    	undefined: [0, 100]
	    };

	    minmax = minmaxMap[currentVar] || [minAvg, maxAvg];
	}

	onMount(() => {
		const mediaQuery = window.matchMedia('(prefers-reduced-motion: reduce)');

		function updatePreference(e) {
			prefersReducedMotion = e.matches;
		}

        // Add listener
		mediaQuery.addListener(updatePreference);

        // Set initial value
		prefersReducedMotion = mediaQuery.matches;

        // Cleanup function
		return () => {
			mediaQuery.removeListener(updatePreference);
		};
	});

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
		if (selectedStandingPct == -1) {
			const indexWithShowYou = copy.story.findIndex(item => item.showyou == 1);
			if (indexWithShowYou !== -1) {
				copy.story.splice(indexWithShowYou, 1);
			}
			showAverageStage = 8;
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
		<Canvas {prefersReducedMotion} {bg} {axis_flip} {x_axis_variable} {axis_variable} {x_axis_variable_range} bind:searchValue {selectedIndices} {data} {copy} questions={currentQuestionOrder} {currentVar} {currentData} {currentQuestionNum} {currentStageNumber} {minmax} {minIndicies} {maxIndicies} {showAverageStage} bind:reset bind:explored/>
		<!-- {#key axis_variable}
		<Axis {currentStageNumber} {axis_flip} {axis_variable} {x_axis_variable} {x_axis_variable_range} />
		{/key} -->
		<Text bind:searchValue {currentStageNumber} {copy} {data} {currentVar} on:updateCurrentVar={handleCurrentVarUpdate} on:updateQuestion={handleUpdateQuestion} bind:introActive  bind:questionNumber bind:reset/>
	</div>
	{/if}
</div>

<style>
	.container {
		position: fixed;
		width: calc(100% - 370px);
		right: 370px;
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
