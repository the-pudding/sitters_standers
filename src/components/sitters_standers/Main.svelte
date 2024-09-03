<script>
	import Canvas from "$components/sitters_standers/Canvas.svelte"; 
	// import Pulldown from "$components/sitters_standers/Pulldown.svelte"; 
	import Question from "$components/sitters_standers/Question.svelte"; 
	export let copy;
	export let data;
	export let currentQuestionNum = 0;

	let currentData = Array(copy.questions.length).fill(1);
	let currentVar = copy.questions[currentQuestionNum].variable;
	let currentQuestionOrder = copy.questions.map(question => question.variable);

	data = data.sort((a, b) => b.TOT_EMP - a.TOT_EMP);
	
	function getPercentKeys(arr) {
		if (arr.length === 0) {
			return [];
		}
		const firstObject = arr[0];
		let percentKeys = Object.keys(firstObject).filter(key => key.startsWith("Percent"));
		percentKeys.push("H_MEAN");
		return percentKeys;
	}

	function handlePulldownChange(event) {
		selectedVar = event.detail.selectedVar;
	}

	function handleUpdateQuestion(event, num) {
		currentData[currentQuestionNum] = event.detail.answer;
		currentQuestionNum += event.detail.direction;
		if (event.detail.direction == -1) {
			currentData[currentQuestionNum] = 1;
		}
		currentVar = copy.questions[currentQuestionNum].variable;
	}

	$: {
		currentVar, currentQuestionNum;
	}
</script>

<div class="container">
	<!-- <Pulldown data={getPercentKeys(data)} {selectedVar} on:change={handlePulldownChange} /> -->
	<Question {currentQuestionNum} {copy} on:updateQuestion={handleUpdateQuestion}  />
	<Canvas {data} {copy} questions={currentQuestionOrder} {currentVar} {currentData} {currentQuestionNum}/>
</div>

<style>
	.container {
		position: absolute;
		width: 100%;
		height: calc(100vh - 124.5px);
	}
</style>
