<script>
	export let currentStageNumber, axis_variable, x_axis_variable_range, x_axis_variable, axis_flip;
	import { fade } from 'svelte/transition';
</script>

{#if currentStageNumber > 0 && axis_variable != -1}
<div class="yaxis" transition:fade>
	{#if axis_flip == 1}
	<div class="top mobile">↓ Less {axis_variable}</div>
	<div class="bottom mobile">↑ More {axis_variable}</div>
	{:else}	
	<div class="top mobile">↑ More {axis_variable}</div>
	<div class="bottom mobile">↓ Less {axis_variable}</div>
	{/if}

</div>
{/if}


<div class="xaxis">
	<!-- {#if currentStageNumber > 1}
	<div class="left mobile" transition:fade>← {x_axis_variable_range[x_axis_variable][2]}</div>
	<div class="right mobile" transition:fade>{x_axis_variable_range[x_axis_variable][3]} →</div>
	{/if} -->
</div>


<style>
	.yaxis, .xaxis {
		opacity:  0.5;
		user-select: none;
		font-size: 14px;
		pointer-events: none;
/* 		color: var(--color-light-purple); */
		color:  white;
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
		right: 10px;
		top: 0px;
		width: calc(100% - 10px);
		pointer-events: none;
		height: 100%;
		z-index: 100;
		text-align: right;
		user-select: none;
	}
	.yaxis .top {
		position: absolute;
		top: 10px;
		right:  0px;
		text-align:  right;
	}
	.yaxis .bottom {
		position: absolute;
		bottom: 10px;
		right:  0px;
		text-align:  right;
	}

	.xaxis {
		position: absolute;
		left: 5px;
		bottom: 30px;
/* 		border-bottom: 1px dotted var(--color-lessdark-purple); */
		width: 100%;
		height: 30px;
		text-align: left;
		opacity:  0;
	}
	@media (width <= 860px) {
		.xaxis {
			opacity:  1;
		}
		.yaxis .bottom {
			bottom:  40px;
		}
		.yaxis .top {
			top: 50px;
		}
		.yaxis, .xaxis {
			font-size:  13px;
		}
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