<svelte:options runes={true} />

<script lang="ts">
	import * as Carousel from '$lib/components/ui/carousel';
	import * as Card from '$lib/components/ui/card';
	import recipes from '$lib/data/recipes.md?raw';

	import { marked } from 'marked';
	import { onMount } from 'svelte';
	import { shuffle } from './shuffle';
	import type { Recipe } from './types';

	// override the markdown renderer for links to add a target _blank
	const markdownRenderer = {
		link(href: string, title: string | null | undefined, text: string) {
			return `<a href="${href}" title="${title}" target="_blank">${text} ↗</a>`;
		}
	};

	marked.use({ renderer: markdownRenderer });

	const parseRecipeMarkdown = (recipe: string) => {
		console.log(recipe)
		// get comma separated strings as array from a line starting with ---INGR: ending with ---
		const ingredients =
			recipe
				.match(/---INGR:([^-]*)---/)?.[1]
				.split(',')
				.map((ingredient) => ingredient.trim()) ?? [];
		console.log(ingredients);
		// remove the ingredients line from the recipe
		recipe = recipe.replace(/---INGR:[^-]*---/, '');
		// get comma separated strings as array from a line starting with ---FIND: ending with ---
		const whereToFind =
			recipe
				.match(/---FIND:([^-]*)---/)?.[1]
				.split(',')
				.map((ingredient) => ingredient.trim()) ?? [];
		// remove the where to find line from the recipe
		recipe = recipe.replace(/---FIND:[^-]*---/, '');

		return {
			ingredients,
			whereToFind,
			html: marked(recipe)
		} as Recipe;
	};
	const allRecipesMarkdown = recipes.split('---SPLIT---');
	const allRecipesAsHtml = allRecipesMarkdown.map(parseRecipeMarkdown);

	let randomRecipes: Recipe[] = $state([]);
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
								{@html recipe.html}
							</div>
						</Card.Content>
						<Card.Footer>
							{recipe.ingredients}
						</Card.Footer>
					</Card.Root>
				</Carousel.Item>
			{/each}
		</Carousel.Content>
	</Carousel.Root>
</main>
