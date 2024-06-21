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
	import { flip } from 'svelte/animate';
	import { quintOut } from 'svelte/easing';
	import { shuffle } from './shuffle';
	import type { Recipe } from './types';
	import type { CarouselAPI } from '$lib/components/ui/carousel/context';

	// override the markdown renderer for links to add a target _blank
	const markdownRenderer = {
		link(href: string, title: string | null | undefined, text: string) {
			return `<a href="${href}" title="${title}" target="_blank">${text} ↗</a>`;
		}
	};

	marked.use({ renderer: markdownRenderer });

	const parseRecipeMarkdown = (recipe: string) => {
		const tagsRegExp = /---TAGS([^-]*)---/i;
		const tags =
			recipe
				.match(tagsRegExp)?.[1]
				.split(',')
				.map((tag) => tag.trim()) ?? [];

		recipe = recipe.replace(tagsRegExp, '');

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
	// find all tags that would reduce the number of currently filtered recipes + the already selected tags
	let usefulTags = $derived.by(() => {
		return filteredTags.filter((tag) => {
			if (selectedTags.includes(tag)) {
				return true;
			}
			const filteredRecipesWithThisTag = filteredRecipes.filter((recipe) =>
				recipe.tags.includes(tag)
			);
			return filteredRecipesWithThisTag.length !== filteredRecipes.length;
		});
	});

	let carouselApi: CarouselAPI | undefined = $state();
	let currentSlideIndex = $state(1);
	$effect(() => {
		const getCurrentSlide = () => {
			currentSlideIndex = (carouselApi && carouselApi.selectedScrollSnap() + 1) || 0;
		};
		if (!!carouselApi) {
			carouselApi.on('select', getCurrentSlide);
			carouselApi.on('slidesChanged', getCurrentSlide);
		}
		return () => {
			if (!!carouselApi) {
				carouselApi.off('select', getCurrentSlide);
				carouselApi.off('slidesChanged', getCurrentSlide);
			}
		};
	});
	let filterSheetOpen = $state(false);
	$inspect(filterSheetOpen);
	onMount(() => {
		const recipes = [...allRecipes];
		shuffle(recipes);
		randomRecipes = recipes;
	});
</script>

<main class="flex h-screen flex-col">
	<Sheet.Root bind:open={filterSheetOpen}>
		<Sheet.Content class="flex flex-col">
			<Sheet.Header class="grow self-start">
				<Sheet.Title class="mb-2">Filter by tags</Sheet.Title>
			</Sheet.Header>
			<div class="my-2 flex flex-wrap gap-2">
				{#each usefulTags as tag (tag)}
					<div animate:flip={{ duration: 500, easing: quintOut }}>
						<Toggle
							class="flex-initial gap-1 text-xs md:text-sm"
							pressed={selectedTags.includes(tag)}
							variant="outline"
							onPressedChange={(pressed) => onFilterTagChange(tag, pressed)}
						>
							<span>{sanitizeTag(tag)}</span>
							<span>({filteredRecipes.filter((recipe) => recipe.tags.includes(tag)).length})</span>
						</Toggle>
					</div>
				{/each}
			</div>
			<Sheet.Footer>
				<div class="flex gap-2 py-4">
					<Button variant="secondary" class="grow" on:click={() => (selectedTags = [])}>
						<FilterX class="mr-2 h-4 w-4" />
						Reset filters
					</Button>
					<Button class="grow" on:click={() => (filterSheetOpen = false)}>
						<Filter class="mr-2 h-4 w-4" />
						Apply
					</Button>
				</div>
			</Sheet.Footer>
		</Sheet.Content>
	</Sheet.Root>

	<Carousel.Root bind:api={carouselApi} class="h-full" opts={{ loop: true }}>
		<Carousel.Content>
			{#each filteredRecipes as recipe}
				<Carousel.Item class="h-full">
					<Card.Root class="m-2">
						<Card.Content>
							<div class="prose">
								<!-- eslint-disable svelte/no-at-html-tags -->
								{@html recipe.html}
							</div>
						</Card.Content>
						<Card.Footer>
							<div class="flex flex-wrap gap-1">
								{#each recipe.tags as tag}
									<Badge class="text flex-initial text-sm" variant="outline">
										{sanitizeTag(tag)}
									</Badge>
								{/each}
							</div>
						</Card.Footer>
					</Card.Root>
				</Carousel.Item>
			{/each}
		</Carousel.Content>
	</Carousel.Root>
</main>
<div class="fixed bottom-0 right-0 m-2 flex items-center gap-4">
	<div>
		<span>{currentSlideIndex} / {filteredRecipes.length}</span>
	</div>
	<Button on:click={() => (filterSheetOpen = true)}>
		<Filter class="mr-2 h-4 w-4" />
		<span>Filters ({selectedTags.length})</span>
	</Button>
</div>
