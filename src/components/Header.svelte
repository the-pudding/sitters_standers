<script>
	// shadow, plain, line, bubble
	import wordmark from "$svg/wordmark-shadow.svg";
	import wordmarksmall from "$svg/logo-white.svg";
	import { fade } from 'svelte/transition';
	export let currentStageNumber, introActive, stage;
</script>

<header>
	{#if introActive}
	<div class="wordmark" transition:fade>
		<a href="https://pudding.cool" aria-label="The Pudding" target="_self"
			>{@html wordmark}</a
		>
	</div>
	{/if}
	{#if stage == "explore"}
		<div class="wordmark small" transition:fade>
		<a href="https://pudding.cool" aria-label="The Pudding" target="_self"
			>{@html wordmarksmall}</a
		>
	</div>
	{/if}
</header>

<style>
	header {
		pointer-events: none;
	}
	.wordmark {
		max-width: 15em;
		margin: 0 auto;
		padding: 15px;
		transform: rotate(-4deg);
	}
	.wordmark.small {
		max-width:  6em;
	}

	.wordmark a {
		border: none;
		display: block;
		color: var(--color-fg);
		pointer-events: auto;
	}

	.wordmark a:hover {
		background-color: transparent;
	}
	@media (width <= 1000px) {
		.wordmark {
			max-width: 10em;
			margin: 0 0 0 10px;
			padding: 20px 0px 0 0;
			transform: rotate(-4deg);
		}
		.wordmark.small {
		max-width:  3em;
	}
	}
</style>
