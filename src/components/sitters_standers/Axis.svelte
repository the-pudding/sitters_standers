<script>
	export let currentStageNumber;
</script>

{#if currentStageNumber > 0}
<div class="yaxis">
	<div class="top">↑ Higher pay</div>
	<div class="bottom">↓ Lower pay</div>
</div>
{/if}

{#if currentStageNumber > 1}
<div class="xaxis">
	<div class="left">← Less like your job</div>
	<div class="right">More like your job →</div>
</div>
{/if}

<style>
	.yaxis, .xaxis {
		user-select: none;
		font-size: 16px;
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
	}
	.yaxis {
		position: absolute;
		left: 5px;
		top: 0px;
		width: 120px;
		pointer-events: none;
		height: 100%;
/*		border-left: 1px solid var(--color-light-purple);*/
		z-index: 100;

	}
	.yaxis .top {
		position: absolute;
		top: 10px;
		left: 5px;
	}
	.yaxis .bottom {
		position: absolute;
		bottom: 40px;
		left: 5px;
	}
	.xaxis {
		position: absolute;
		left: 5px;
		bottom: 30px;
		border-bottom: 1px solid var(--color-light-purple);
		width: 100%;
		height: 30px;
	}
	.xaxis .left {
		position: absolute;
		left: 5px;
		bottom: -25px;
		width: 150px;
	}
	.xaxis .right {
		position: absolute;
		right: 10px;
		bottom: -25px;
		width: 170px;
		text-align: right;
	}
</style>