<svelte:options runes={true} />

<script lang="ts">
	import * as Carousel from '$lib/components/ui/carousel';
	import * as Card from '$lib/components/ui/card';
	import * as Sheet from '$lib/components/ui/sheet';
	import { Toggle } from '$lib/components/ui/toggle';
	import { Button } from '$lib/components/ui/button';
	import { Settings2 } from 'lucide-svelte';
	import { Badge } from '$lib/components/ui/badge';
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
		const tags =
			recipe
				.match(/---TAGS([^-]*)---/)?.[1]
				.split(',')
				.map((tag) => tag.trim()) ?? [];
		recipe = recipe.replace(/---TAGS[^-]*---/, '');

		return {
			tags,
			html: marked(recipe)
		} as Recipe;
	};
	const allRecipesMarkdown = recipes.split('---SPLIT---');
	const allRecipes = allRecipesMarkdown.map(parseRecipeMarkdown);
	const allTags = new Set(allRecipes.flatMap((recipe) => recipe.tags));
	const allTagsSorted = [...allTags].toSorted((a, b) => {
		const aIsSpecial = a.match(/^[A-Z]_/) ? 1 : 0;
		const bIsSpecial = b.match(/^[A-Z]_/) ? 1 : 0;
		return bIsSpecial - aIsSpecial || a.localeCompare(b);
	});
	const sanitizeTag = (tag: string) => tag.replace(/^[A-Z]_/, '');
	const onFilterTagChange = (tag: string, pressed: boolean) => {
		if (pressed) {
			selectedTags.push(tag);
		} else {
			const indexOfTag = selectedTags.indexOf(tag);
			if (indexOfTag !== -1) {
				selectedTags.splice(indexOfTag, 1);
			}
		}
	};

	let randomRecipes: Recipe[] = $state([]);
	let selectedTags: string[] = $state([]);
	let filteredRecipes = $derived.by(() => {
		if (selectedTags.length === 0) {
			return randomRecipes;
		}
		return randomRecipes.filter((recipe) => selectedTags.every((tag) => recipe.tags.includes(tag)));
	});
	onMount(() => {
		const recipes = [...allRecipes];
		shuffle(recipes);
		randomRecipes = recipes;
	});
</script>

<main class="h-full p-2">
	<Sheet.Root>
		<Sheet.Trigger class="mb-3">
			<Button>
				<Settings2 class="mr-2 h-4 w-4" />
				Filters ({selectedTags.length})
			</Button>
		</Sheet.Trigger>
		<Sheet.Content>
			<Sheet.Header>
				<Sheet.Title>Filter by tags</Sheet.Title>
			</Sheet.Header>
			<div class="grid grid-flow-row grid-cols-3 gap-4">
				{#each allTagsSorted as tag (tag)}
					<Toggle
						pressed={selectedTags.includes(tag)}
						variant="outline"
						onPressedChange={(pressed) => onFilterTagChange(tag, pressed)}
					>
						{sanitizeTag(tag)}
					</Toggle>
				{/each}
			</div>
		</Sheet.Content>
	</Sheet.Root>

	<Carousel.Root opts={{ loop: true }}>
		<Carousel.Content>
			{#each filteredRecipes as recipe}
				<Carousel.Item class="h-screen">
					<Card.Root>
						<Card.Content>
							<div class="prose p-1">
								<!-- eslint-disable svelte/no-at-html-tags -->
								{@html recipe.html}
							</div>
						</Card.Content>
						<Card.Footer>
							<div class="flex flex-wrap gap-1">
								{#each recipe.tags as tag}
									<Badge class="flex-initial" variant="outline">{sanitizeTag(tag)}</Badge>
								{/each}
							</div>
						</Card.Footer>
					</Card.Root>
				</Carousel.Item>
			{/each}
		</Carousel.Content>
	</Carousel.Root>
</main>
