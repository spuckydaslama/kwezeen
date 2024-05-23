<svelte:options runes={true} />

<script lang="ts">
	import * as Carousel from '$lib/components/ui/carousel';
	import * as Card from '$lib/components/ui/card';
	import * as Sheet from '$lib/components/ui/sheet';
	import { Toggle } from '$lib/components/ui/toggle';
	import { Button } from '$lib/components/ui/button';
	import { Filter, FilterX } from 'lucide-svelte';
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
	let filteredTags = $derived.by(() => {
		// filter out tags that are not in the filtered recipes
		const tags = new Set(filteredRecipes.flatMap((recipe) => recipe.tags));
		return allTagsSorted.filter((tag) => tags.has(tag));
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
				<Filter class="mr-2 h-4 w-4" />
				Filters ({selectedTags.length})
			</Button>
		</Sheet.Trigger>
		<Sheet.Content>
			<Sheet.Header>
				<Sheet.Title class="mb-2">Filter by tags</Sheet.Title>
			</Sheet.Header>
			<div class="flex flex-wrap gap-1 my-2">
				{#each filteredTags as tag (tag)}
					<Toggle
						class="flex-initial"
						pressed={selectedTags.includes(tag)}
						variant="outline"
						onPressedChange={(pressed) => onFilterTagChange(tag, pressed)}
					>
						{sanitizeTag(tag)}
					</Toggle>
				{/each}
			</div>
			<Sheet.Footer>
				<Button onclick={() => selectedTags = []}>
					<FilterX class="mr-2 h-4 w-4" />
					Reset filters
				</Button>
			</Sheet.Footer>
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
