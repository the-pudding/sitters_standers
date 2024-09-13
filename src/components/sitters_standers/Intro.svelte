<script>
  export let copy;
  export let selectedIndices;
  export let introActive;

  function toggleAnswer(index) {
    if (selectedIndices.includes(index)) {
      selectedIndices = selectedIndices.filter(i => i !== index);
    } else {
      selectedIndices = [...selectedIndices, index];
    }
  }

  function introDone() {
  	introActive = false;
  }
</script>

<div class="introContainer">
  <div class="textContainer">
    <div class="questionContainer">
      {@html copy.first_question[0].text}
    </div>
    <div class="answerContainer">
      {#each copy.first_question[0].options as option, i}
        <button 
          class="answerItem {selectedIndices.includes(i) ? 'selected' : ''}" 
          on:click={() => toggleAnswer(i)}>
          {option}
        </button>
      {/each}
    </div>
    <button class="ready" on:click={introDone}>Let's go</button>
  </div>
</div>
<style>
	h1 {
		font-size: 20px;
	}
	h2 {
		font-size: 16px;
	}
	.introContainer {
		background: rgba(255,255,255,0.9);
		position: fixed;
		left: 0;
		top: 0;
		width: 100%;
		height: 100%;
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
	.questionContainer {
		font-size: 15px;
		margin-bottom: 10px;
	}
	.answerItem {
		font-size: 15px;
		display: block;
		position: relative;
		width: 100%;
		padding: 10px 0;
		background: white;
		border: 1px solid #ccc;
		margin-top: -1px;
		z-index: 1;
	}
	.answerItem:hover {
		border: 1px solid #000;
		z-index: 2;
	}
	.answerItem.selected {
		font-weight: bold;
	}
	.answerItem.selected::before {
		content: "✅ ";
	}
	.ready {
		width: 100%;
		margin-top: 20px;
	}
</style>