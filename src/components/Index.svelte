<script>
	import { getContext } from "svelte";
	import Sitters_Standers from "$components/sitters_standers/Main.svelte";
	// import Footer from "$components/Footer.svelte";

	const copy = getContext("copy");
	copy.questions = copy.questions.map((question, index) => ({
		...question,
		index: index
	}));
	copy.first_question[0].options = copy.questions
		.filter(question => question.excluded !== "1")
		.map(question => question.short);
	const data = getContext("data");
</script>

<Sitters_Standers {copy} {data}/>
<!-- <Footer /> -->
