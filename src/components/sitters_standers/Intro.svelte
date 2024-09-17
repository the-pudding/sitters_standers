<script>
	export let copy;
	export let selectedIndices;
	export let introActive;

	let shown = true;
	function toggleAnswer(index) {
		if (selectedIndices.includes(index)) {
			selectedIndices = selectedIndices.filter(i => i !== index);
		} else {
			selectedIndices = [...selectedIndices, index];
		}
	}

	function toggleShown() {
		shown = !shown;
	}

	function introDone() {
		introActive = false;
		shown = false;
	}

	$: {
		selectedIndices, introActive;
	}
</script>

{#if shown}
<div class="introContainer" class:introInactive={!introActive}>
	<div class="textContainer">
	{#if introActive}
		<div class="questionContainer">
			{@html copy.first_question[0].text}
		</div>
		{/if}
		<div class="answerContainer">
			{#each copy.first_question[0].options as option, i}
			<button 
			class="answerItem {selectedIndices.includes(i) ? 'selected' : ''}" 
			on:click={() => toggleAnswer(i)}>
			{option}
		</button>
		{/each}
	</div>
	{#if introActive}
	<button class="ready" on:click={introDone}>Let's go</button>
	{/if}
</div>
</div>
{/if}


{#if !introActive}
<button class="hideShow" on:click={toggleShown}>
	{#if shown}
	x Close job requirements
	{:else}
	✎ Edit job requirements
	{/if}
</button>
{/if}

<style>
	
	.introContainer {
		background: #211d21;
		position: fixed;
		color: white;
		left: 0;
		top: 0;
		width: 100%;
		height: 100%;
		z-index: 10000;
		font-size: 14px;
	}
	.introInactive.introContainer {
		background: none;
		position: fixed;
		left: auto;
		right: 0;
		top: 55px;
		width: 300px;
		height: auto;
		padding: 10px;
	}
	.textContainer {
		max-width: 300px;
		width: 100%;
		text-align: center;
		position: absolute;
		left: 50%;
		top: 50%;
		transform: translate(-50%, -50%);
	}
	.introInactive .textContainer {
		position: relative;
		width: 100%;
		left: auto;
		right: 0;
		top: 0px;
		transform: translate(0%, 0%);
	}
	.questionContainer {
		margin-bottom: 10px;
	}
	.answerItem {
		display: block;
		position: relative;
		width: 100%;
		padding: 10px 0;
		background: #141314;
		border: 1px solid #473847;
		color: #e0abe0;
		margin-top: -1px;
		z-index: 1;
	}
	.answerItem:hover {
		border: 1px solid #755475;
		z-index: 2;
	}
	.answerItem.selected {
		background: #f371f5;
		color: black;
	}
	.answerItem.selected::before {
		content: "✔ ";
		position: absolute;
		width: 20px;
		left: 0px;
		text-align: center;
		top: 50%;
		transform: translateY(-50%);
	}
	.ready {
		width: 100%;
		margin-top: 20px;
		background: #e6d8e8;
		color: #211d21;
	}
	.ready:hover {
		text-decoration: underline;
	}
	.hideShow {
		position: fixed;
		right: 20px;
		top: 20px;
		color: white;
		background: #141314;
		border: 1px solid #473847;
		cursor: pointer;
		font-size: 14px;
		z-index: 10001;
	}
	.hideShow:hover {
		border: 1px solid #755475;
	}
</style>