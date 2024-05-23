<script lang="ts">
	import * as Carousel from '$lib/components/ui/carousel';
	import recipes from '$lib/data/recipes.md?raw';

	import { marked } from 'marked';
	import {onMount} from "svelte";
	import {shuffle} from "./shuffle";

	const allRecipesMarkdown = recipes.split('---SPLIT---');
	const allRecipesAsHtml = allRecipesMarkdown.map((recipeMarkdown) => marked(recipeMarkdown) as string);

	let randomRecipes: string[] = $state([]);
	onMount(() => {
		const recipes = [...allRecipesAsHtml];
		shuffle(recipes);
		randomRecipes = recipes;
	})
</script>

<main>
	<script lang="ts">
	</script>

	<Carousel.Root class="w-full" opts={{loop: true}}>
		<Carousel.Content>
			{#each randomRecipes as recipe}
				<Carousel.Item>
					<div class="p-1 prose border-amber-600 border-2 border-solid">
						{@html recipe}
					</div>
				</Carousel.Item>
			{/each}
		</Carousel.Content>
	</Carousel.Root>
</main>
