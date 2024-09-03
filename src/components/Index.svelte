<script>
	import { getContext } from "svelte";
	import Sitters_Standers from "$components/sitters_standers/Main.svelte";
	// import Footer from "$components/Footer.svelte";

	const copy = getContext("copy");
	copy.questions = copy.questions.map((question, index) => ({
		...question,
		num: index + 1
	}));
	const data = getContext("data");
</script>

<Sitters_Standers {copy} {data}/>
<!-- <Footer /> -->
