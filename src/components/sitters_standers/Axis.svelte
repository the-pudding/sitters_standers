<script>
	export let currentStageNumber, axis_variable;
	import { fade } from 'svelte/transition';
</script>

{#if currentStageNumber > 0}
<div class="yaxis" transition:fade>
<!-- 	<div class="top desktop">↑ Higher pay</div>
	<div class="bottom desktop">↓ Lower pay</div> -->
	<div class="top mobile">↑ More {axis_variable}</div>
	<div class="bottom mobile">↓ Less {axis_variable}</div>
</div>
{/if}


<div class="xaxis">
	{#if currentStageNumber > 1}
<!-- 	<div class="left desktop" transition:fade>← Less like your job</div>
	<div class="right desktop" transition:fade>More like your job →</div> -->
	<div class="left mobile" transition:fade>← Lower pay</div>
	<div class="right mobile" transition:fade>Higher pay →</div>
	{/if}
</div>


<style>
	.yaxis, .xaxis {
		font-family: var(--serif);
		user-select: none;
		font-size: 13px;
		pointer-events: none;
		color: var(--color-light-purple);
		text-shadow: 
		1px 1px 0px var(--color-bg), /* Bottom-right shadow */
		-1px -1px 0px var(--color-bg), /* Top-left shadow */
		-1px 1px 0px var(--color-bg), /* Bottom-left shadow */
		1px -1px 0px var(--color-bg), /* Top-right shadow */
		2px 2px 0px var(--color-bg), /* Extra width for outline */
		-2px -2px 0px var(--color-bg),
		-2px 2px 0px var(--color-bg),
		2px -2px 0px var(--color-bg);
		z-index:9999;
	}

	.yaxis {
		position: absolute;
		right: 0px;
		top: 0px;
		width: 200px;
		pointer-events: none;
		height: 100%;
		z-index: 100;
		text-align: right;
	}
	.yaxis .top {
		position: absolute;
		top: 60px;
		right: 5px;
	}
	.yaxis .bottom {
		position: absolute;
		bottom: 40px;
		right: 5px;
	}

	.xaxis {
		position: absolute;
		left: 5px;
		bottom: 30px;
		border-bottom: 1px dotted var(--color-lessdark-purple);
		width: 100%;
		height: 30px;
		text-align: left;
	}

	.xaxis .left {
		position: absolute;
		bottom: -28px;
		width: 150px;
	}
	.xaxis .right {
		position: absolute;
		right: 10px;
		bottom: -28px;
		width: 170px;
		text-align: right;
	}

	.yaxis div {
		width: 100%;
	}
	.yaxis .bottom {
		bottom: 35px;
	}
	.yaxis .top {
/*			top: 100px;*/
}
.xaxis .left, .xaxis .right {
	bottom: -20px;
}
/*.mobile {
	display: none;
}
.desktop {
	display: block;
}*/
/*@media (width <= 800px) {*/
	.mobile {
		display: block;
	}
	.desktop {
		display: none;
	}
/*}*/
</style>