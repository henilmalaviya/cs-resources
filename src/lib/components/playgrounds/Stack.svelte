<script lang="ts">
	import { cn } from '$lib/utils/shadcn';
	import { Button } from '$lib/components/ui/button';
	import { fade, slide } from 'svelte/transition';
	import { onMount } from 'svelte';

	const MAX = 5;
	const IN_OUT_ANIMATION_DURATION = 400;

	let stack = $state<string[]>([]);
	let TOS: number = $state(-1);
	const isEmpty = $derived(TOS === -1);
	const isFull = $derived(stack.length >= MAX);

	const operations = $state<string[]>([]);

	const limitedOperations = $derived(operations.slice(-6));

	function push() {
		if (isFull) {
			return;
		}
		operations.push(`TOS++ (${TOS} ---> ${TOS + 1})`);
		TOS++;
		const element = String.fromCharCode(65 + TOS);
		stack.push(element);
		operations.push(`Stack[${TOS}] = ${element}`);
	}

	function pop() {
		if (isEmpty) {
			return;
		}
		stack.pop();
		operations.push(`TOS-- (${TOS} ---> ${TOS - 1})`);
		TOS--;
	}

	onMount(() => {
		// push();
	});
</script>

<div class="playground">
	<!-- stack -->
	<div class={cn('stack-box', isFull && 'full', isEmpty && 'empty')}>
		{#each stack as item, i}
			{@const isHead = TOS === i}
			<!-- svelte-ignore a11y_click_events_have_key_events -->
			<!-- svelte-ignore a11y_no_static_element_interactions -->
			<div
				in:slide|local={{ duration: IN_OUT_ANIMATION_DURATION }}
				out:slide|local={{ duration: IN_OUT_ANIMATION_DURATION }}
				class={cn('stack-item', isHead && 'head')}
				onclick={pop}
			>
				{item}
				<span class="index">
					{i}
				</span>
			</div>
		{/each}
	</div>
	<div class="stack-actions">
		<Button disabled={isFull} onclick={push}>Push</Button>
		<Button disabled={isEmpty} onclick={pop}>Pop</Button>
	</div>

	<div class="stack-info">
		<div class="core-info">
			<div class="flex gap-2">
				<span class="font-semibold">TOS Index:</span>
				<span>{TOS}</span>
			</div>
			<div class="flex gap-2">
				<span class="font-semibold">TOS Element:</span>
				<span>{stack[TOS] || 'NULL'}</span>
			</div>
			<div class="flex gap-2">
				<span class="font-semibold">isFull:</span>
				<span>{isFull ? 'Yes' : 'No'}</span>
			</div>
			<div class="flex gap-2">
				<span class="font-semibold">isEmpty:</span>
				<span>{isEmpty ? 'Yes' : 'No'}</span>
			</div>
		</div>
		<div class="operations">
			<h4 class="border-b">Operations</h4>
			{#if limitedOperations.length === 0}
				<span class="text-muted-foreground italic text-sm">No operations yet.</span>
			{/if}
			{#each limitedOperations as operation, i}
				{@const isLast = i === limitedOperations.length - 1}
				<div in:slide|local class={cn('operation', isLast && 'font-semibold')}>
					{operation}
					{#if isLast}
						<span class="text-xs italic text-muted-foreground ml-8">(Most Recent)</span>
					{/if}
				</div>
			{/each}
		</div>
	</div>
</div>

<style lang="postcss">
	.playground {
		width: 100%;
		height: fit-content;

		padding: theme('padding.0') theme('padding.0');

		display: flex;
		gap: theme('gap.8');
	}

	.stack-box {
		width: theme('width.40');
		height: 100%;
		min-height: theme('minHeight.72');

		padding: theme('padding.2');

		border-radius: theme('borderRadius.lg');
		border-top-left-radius: 0;
		border-top-right-radius: 0;

		border: 1px solid theme('borderColor.zinc.400');
		border-top: none;

		display: flex;
		flex-direction: column-reverse;
		gap: theme('gap.1');

		transition: all 0.3s linear;
		position: relative;

		--stack-item-height: theme('height.12');

		& .stack-item {
			width: 100%;
			height: var(--stack-item-height);

			transition: all 0.2s ease-in-out;

			background-color: theme('backgroundColor.foreground');

			border-radius: theme('borderRadius.md');

			font-size: theme('fontSize.lg');
			color: theme('textColor.background');

			text-align: center;

			padding: theme('padding.2') 0;
			position: relative;

			& .index {
				position: absolute;
				bottom: theme('space.1');
				left: theme('space.1');
				font-size: theme('fontSize.xs');
				font-weight: theme('fontWeight.bold');
			}

			&.head {
				cursor: pointer;
				position: relative;

				&:hover {
					background-color: theme('backgroundColor.red.600');
				}

				&::after {
					content: 'TOS';
					position: absolute;
					top: theme('space.1');
					right: theme('space.1');
					font-size: theme('fontSize.xs');
					font-weight: theme('fontWeight.bold');
				}
			}
		}

		&.full {
			background-color: theme('backgroundColor.red.200');
			border-color: theme('borderColor.red.600');
		}

		&.empty {
			background-color: theme('backgroundColor.green.200');
			border-color: theme('borderColor.green.600');

			&::after {
				content: 'I am Empty';
				position: absolute;
				top: 50%;
				left: 50%;
				transform: translate(-50%, -50%);
				font-size: theme('fontSize.xs');
				font-weight: theme('fontWeight.semibold');
			}
		}
	}

	.stack-actions {
		width: fit-content;
		height: 100%;

		padding: theme('padding.2') theme('padding.2');

		display: flex;
		flex-direction: column;

		gap: theme('gap.2');
	}

	.stack-info {
		display: flex;
		flex-direction: column;

		gap: theme('padding.4');

		& .core-info {
			@apply grid-cols-2;
			display: grid;
			gap: theme('padding.1') theme('padding.16');
		}

		& .operations {
			& .operation {
				font-size: theme('fontSize.sm');
				color: theme('colors.black');
			}
		}
	}
</style>
