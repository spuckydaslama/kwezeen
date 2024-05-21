<script lang="ts">
	// import recipes.md and split the text by '---'
	import recipes from '$lib/data/recipes.md?raw';

	const allRecipes = recipes.split('---SPLIT---');
</script>

<main>
	{#each allRecipes as recipe}
		<pre>{recipe}</pre>
	{/each}
</main>
