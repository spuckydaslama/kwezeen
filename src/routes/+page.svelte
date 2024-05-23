<svelte:options runes={true} />

<script lang="ts">
	import * as Carousel from '$lib/components/ui/carousel';
	import * as Card from '$lib/components/ui/card';
	import recipes from '$lib/data/recipes.md?raw';

	import { marked } from 'marked';
	import { onMount } from 'svelte';
	import { shuffle } from './shuffle';

	const allRecipesMarkdown = recipes.split('---SPLIT---');
	const allRecipesAsHtml = allRecipesMarkdown.map(
		(recipeMarkdown) => marked(recipeMarkdown) as string
	);

	let randomRecipes: string[] = $state([]);
	onMount(() => {
		const recipes = [...allRecipesAsHtml];
		shuffle(recipes);
		randomRecipes = recipes;
	});
</script>

<main>
	<Carousel.Root opts={{ loop: true }}>
		<Carousel.Content>
			{#each randomRecipes as recipe}
				<Carousel.Item class="h-screen">
					<Card.Root class="m-2">
						<Card.Content>
							<div class="prose p-1">
								<!-- eslint-disable svelte/no-at-html-tags -->
								{@html recipe}
							</div>
						</Card.Content>
					</Card.Root>
				</Carousel.Item>
			{/each}
		</Carousel.Content>
	</Carousel.Root>
</main>
