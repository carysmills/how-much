<script>
	import Intro from './Intro.svelte';
	import List from './List.svelte';
	import Receipt from './Receipt.svelte';
</script>

<div>
	<Intro />
	<main>
		<List />
		<Receipt />
	</main>
</div>

<style>
	main {
		display: flex;
		justify-content: space-around;
	}

	@media only screen and (max-width: 768px) {
		main {
			flex-direction: column;
		}
	}
</style>
