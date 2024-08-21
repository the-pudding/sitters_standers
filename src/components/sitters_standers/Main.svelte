<script>
	export let copy;
	export let data;
	data = data.sort((a, b) => b.TOT_EMP - a.TOT_EMP);
	import Canvas from "$components/sitters_standers/Canvas.svelte"; 
	
</script>

<div class="container">
	<Canvas {data}/>
</div>

<style>
	.container {
		position: absolute;
		width: 100%;
		height: calc(100vh - 124.5px);
	}
</style>