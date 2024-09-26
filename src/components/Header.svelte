<script>
	// shadow, plain, line, bubble
	import wordmark from "$svg/wordmark-shadow.svg";
</script>

<header>
	<div class="wordmark">
		<a href="https://pudding.cool" aria-label="The Pudding" target="_self"
			>{@html wordmark}</a
		>
	</div>
</header>

<style>
	header {
		pointer-events: none;
	}
	.wordmark {
		max-width: 14em;
		margin: 0 auto;
		padding: 32px 0;
		transform: rotate(-4deg);
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
/*	@media (width <= 1000px) {*/
		.wordmark {
			max-width: 10em;
			margin: 0 0 0 20px;
			padding: 32px 0;
			transform: rotate(-4deg);
		}
/*	}*/
</style>
