<script lang="ts">
	import recipes from '$lib/data/recipes.md?raw';
	import {marked} from "marked";

	const allRecipes = recipes.split('---SPLIT---');
	const recipesHtml = allRecipes.map(recipe => marked(recipe));
</script>

<main>
	{#each recipesHtml as recipe}
		{@html recipe}
	{/each}
</main>
