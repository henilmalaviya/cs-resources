<script lang="ts">
	import type { Snippet } from 'svelte';
	import { Button } from '$lib/components/ui/button';
	import { ArrowsInSimple, ArrowsOutSimple } from 'phosphor-svelte';
	import { cn } from '$lib/utils/shadcn';

	type Props = {
		children: Snippet;
		title?: string;
		isFullScreen?: boolean;
		scrollPosition?: number;
		actions?: Snippet;
		id?: string;
	};

	let {
		children,
		actions,
		title = 'Playground',
		isFullScreen = $bindable(false),
		scrollPosition = $bindable(0),
		id
	}: Props = $props();

	export function togglePlaygroundFullScreen() {
		if (!isFullScreen) {
			// entering
			scrollPosition = window.scrollY;
		} else {
			// exiting
			requestAnimationFrame(() => {
				window.scrollTo(0, scrollPosition);
			});
		}

		isFullScreen = !isFullScreen;
	}
</script>

{#snippet Playground()}
	<div
		{id}
		class={cn(
			'px-4 py-4 border-2 border-dashed flex flex-col gap-2 lg:gap-4',
			isFullScreen && 'border-0 p-0'
		)}
	>
		<div class="flex justify-between">
			<div class="flex gap-2 items-end relative">
				<h3 class="m-0">{title}</h3>
				{#if isFullScreen}
					<div
						class="rounded-full lg:hidden absolute inset-0 w-fit h-fit -top-4 opacity-90 p-0.5 bg-foreground text-background px-2 text-xs"
					>
						Fullscreen Mode
					</div>
				{/if}
			</div>
			<div class="flex gap-2">
				{#if actions}
					{@render actions()}
				{/if}
				<Button
					class="text-lg gap-1"
					size="sm"
					variant={isFullScreen ? 'destructive' : 'secondary'}
					onclick={togglePlaygroundFullScreen}
				>
					{#if isFullScreen}
						<ArrowsInSimple class="text-xl" />
						<span class="text-sm hidden lg:block">Exit Fullscreen</span>
					{:else}
						<ArrowsOutSimple class="text-xl" />
					{/if}
				</Button>
			</div>
		</div>
		<hr class="m-0 mb-2" />

		{@render children()}
	</div>
{/snippet}

{@render Playground()}
