<script>
	export let currentStageNumber, axis_variable, x_axis_variable_range, x_axis_variable, axis_flip;
	import { fade } from 'svelte/transition';
</script>

{#if currentStageNumber > 0 && axis_variable != -1}
<div class="yaxis" transition:fade>
	<div class="top">↑ More {axis_variable}</div>
	<div class="bottom">↓ Less {axis_variable}</div>
</div>
{/if}


<style>
	.yaxis, .xaxis {
		opacity:  0.8;
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
		left: 10px;
		top: 50%;
		width: calc(100% - 10px);
		pointer-events: none;
		height: 100%;
		z-index: 100;
		text-align: left;
		user-select: none;
	}
	.yaxis .top {
		position: absolute;
		top: 10px;
		left:  0px;
		text-align:  left;
	}
	.yaxis .bottom {
		position: absolute;
		bottom: 10px !important;
		left:  0px;
		text-align:  left;
	}

	.xaxis {
		position: absolute;
		left: 5px;
		bottom: 30px;
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
			bottom:  40px !important;
		}
		.yaxis .top {
			top:  auto;
			bottom: 60px;
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