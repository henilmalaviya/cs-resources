<script lang="ts">
	import BackArrowTitle from '$lib/components/BackArrowTitle.svelte';
	import CodeBlock from '$lib/components/CodeBlock.svelte';
	import { CONST } from '$lib/const';
	import { ArrowsInSimple, ArrowsOutSimple } from 'phosphor-svelte';
	import PageUnderDevelopment from '$lib/components/PageUnderDevelopment.svelte';
	import { cn } from '$lib/utils/shadcn';
	import InfixToPostfixPlayground from '$lib/components/playgrounds/InfixToPostfix.svelte';
	import { Button } from '$lib/components/ui/button';
	import PlaygroundContainer from '$lib/components/shared/PlaygroundContainer.svelte';
	import PlaygroundFullScreenOverlay from '$lib/components/shared/PlaygroundFullScreenOverlay.svelte';

	let infixToPostfixPlaygroundRef: ReturnType<typeof InfixToPostfixPlayground> | undefined =
		$state();

	let isPlaygroundFullScreen = $state(false);
	let scrollPosition = $state(0);
</script>

{#snippet Playground()}
	{#snippet Actions()}{/snippet}

	<PlaygroundContainer
		actions={Actions}
		bind:scrollPosition
		bind:isFullScreen={isPlaygroundFullScreen}
		id="playground"
	>
		<InfixToPostfixPlayground bind:this={infixToPostfixPlaygroundRef} />
	</PlaygroundContainer>
{/snippet}

{#if isPlaygroundFullScreen}
	<PlaygroundFullScreenOverlay>
		{@render Playground()}
	</PlaygroundFullScreenOverlay>
{/if}

<div class={cn('prose min-w-full')}>
	<BackArrowTitle href={CONST.ROUTES.DS()._()} title="Infix to Postfix" />

	<PageUnderDevelopment />

	<div class="my-8"></div>

	{@render Playground()}
</div>
