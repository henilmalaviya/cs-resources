<script lang="ts">
	import BackArrowTitle from '$lib/components/BackArrowTitle.svelte';
	import CodeBlock from '$lib/components/CodeBlock.svelte';
	import { CONST } from '$lib/const';
	import { ArrowsInSimple, ArrowsOutSimple } from 'phosphor-svelte';
	import PageUnderDevelopment from '$lib/components/PageUnderDevelopment.svelte';
	import { cn } from '$lib/utils/shadcn';
	import InfixToPostfixPlayground from '$lib/components/playgrounds/InfixToPostfix.svelte';
	import { Button } from '$lib/components/ui/button';

	let infixToPostfixPlaygroundRef: ReturnType<typeof InfixToPostfixPlayground> | undefined =
		$state();

	let isPlaygroundFullScreen = $state(false);
	let scrollPosition = $state(0);

	function togglePlaygroundFullScreen() {
		if (!isPlaygroundFullScreen) {
			// entering
			scrollPosition = window.scrollY;
		} else {
			// exiting
			setTimeout(() => {
				window.scrollTo(0, scrollPosition);
			}, 0);
		}

		isPlaygroundFullScreen = !isPlaygroundFullScreen;
	}
</script>

{#snippet Playground()}
	<div class="flex justify-between">
		<h3 class="m-0">Playground</h3>
		<div class="flex gap-2">
			<Button class="lg:hidden text-lg" size="sm" onclick={togglePlaygroundFullScreen}>
				{#if isPlaygroundFullScreen}
					<ArrowsInSimple />
				{:else}
					<ArrowsOutSimple />
				{/if}
			</Button>
		</div>
	</div>
	<hr class="m-0" />

	<InfixToPostfixPlayground bind:this={infixToPostfixPlaygroundRef} />
{/snippet}

<div class={cn('prose min-w-full')}>
	<BackArrowTitle href={CONST.ROUTES.DS()._()} title="Infix to Postfix" />

	<PageUnderDevelopment />

	<div class="my-8"></div>

	<div class="px-4 py-4 border-2 border-dashed flex flex-col gap-4 min-h-fit">
		{@render Playground()}
	</div>
</div>
