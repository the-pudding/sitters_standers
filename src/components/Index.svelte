<script>
	import { getContext } from "svelte";
	import Header from "$components/Header.svelte";
	import Sitters_Standers from "$components/sitters_standers/Main.svelte";
	// import Footer from "$components/Footer.svelte";

	const copy = getContext("copy");
	let x_axis_variable = "PCT_STAND";
	let introActive = true;
	let currentStageNumber = 0;
	copy.questions = copy.questions.map((question, index) => ({
		...question,
		index: index
	}));
	const data = getContext("data");

	// Filter out data where x_axis_variable is undefined or null
	const filteredData = data.filter(d => d[x_axis_variable] != "");

	$: currentStageNumber;
</script>

<Header {currentStageNumber} {introActive}/>
<Sitters_Standers {copy} data={filteredData} {x_axis_variable} bind:introActive bind:currentStageNumber />