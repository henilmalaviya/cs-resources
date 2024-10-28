<script lang="ts">
	import BackArrowTitle from '$lib/components/BackArrowTitle.svelte';
	import CodeBlock from '$lib/components/CodeBlock.svelte';
	import { CONST } from '$lib/const';
	import { LinkSimple, Check } from 'phosphor-svelte';
	import PageUnderDevelopment from '$lib/components/PageUnderDevelopment.svelte';
	import { cn } from '$lib/utils/shadcn';
	import InfixToPostfixPlayground from '$lib/components/playgrounds/InfixToPostfix.svelte';
	import { Button } from '$lib/components/ui/button';
	import PlaygroundContainer from '$lib/components/shared/PlaygroundContainer.svelte';
	import PlaygroundFullScreenOverlay from '$lib/components/shared/PlaygroundFullScreenOverlay.svelte';
	import { page } from '$app/stores';
	import { copyTextToClipboard } from '$lib/utils/copy.browser';

	let infixToPostfixPlaygroundRef: ReturnType<typeof InfixToPostfixPlayground> | undefined =
		$state();

	let infixExpression = $state('(a + b) * c');
	let infixPlaygroundLoading = $state(true);

	$effect(() => {
		infixPlaygroundLoading = true;
		const playgroundInfixQuery = $page.url.searchParams.get('infix');

		if (infixToPostfixPlaygroundRef && playgroundInfixQuery) {
			infixExpression = decodeURIComponent(playgroundInfixQuery);
			infixToPostfixPlaygroundRef.setInfix(infixExpression);
			infixToPostfixPlaygroundRef.skipAllSteps();
		}

		infixPlaygroundLoading = false;
	});

	let isPlaygroundFullScreen = $state(false);
	let scrollPosition = $state(0);
	let playgroundActionCopied = $state(false);
</script>

{#snippet Playground()}
	{#snippet Actions()}
		<Button
			size="sm"
			class="text-lg"
			onclick={async () => {
				await copyTextToClipboard(`${$page.url.href}?infix=${encodeURIComponent(infixExpression)}`);
				playgroundActionCopied = true;

				setTimeout(() => {
					playgroundActionCopied = false;
				}, 2000);
			}}
		>
			{#if playgroundActionCopied}
				<Check />
			{:else}
				<LinkSimple />
			{/if}
		</Button>
	{/snippet}

	<PlaygroundContainer
		actions={Actions}
		bind:scrollPosition
		bind:isFullScreen={isPlaygroundFullScreen}
		id="playground"
	>
		<InfixToPostfixPlayground
			bind:infix={infixExpression}
			bind:loading={infixPlaygroundLoading}
			bind:this={infixToPostfixPlaygroundRef}
		/>
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
