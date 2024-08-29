<script>
	import Canvas from "$components/sitters_standers/Canvas.svelte"; 
	import Pulldown from "$components/sitters_standers/Pulldown.svelte"; 
	export let copy;
	export let data;
	let selectedVar = getPercentKeys(data)[0];

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
</script>

<div class="container">
	<Pulldown data={getPercentKeys(data)} {selectedVar} on:change={handlePulldownChange} />
	<Canvas {data} {selectedVar}/>
</div>

<style>
	.container {
		position: absolute;
		width: 100%;
		height: calc(100vh - 124.5px);
	}
</style>
