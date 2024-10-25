<script lang="ts">
	import { cn } from '$lib/utils/shadcn';
	import { Button } from '$lib/components/ui/button';
	import { fly, slide } from 'svelte/transition';
	import { ArrowBendLeftDown, ArrowBendUpLeft } from 'phosphor-svelte';

	const MAX = 5;
	const IN_OUT_ANIMATION_DURATION = 300;

	type Props = {};

	const {}: Props = $props();

	let stack = $state<string[]>([]);
	let TOS: number = $state(-1);
	const isEmpty = $derived(TOS === -1);
	const isFull = $derived(stack.length >= MAX);

	let operations = $state<string[]>([]);

	const limitedOperations = $derived(operations.slice(-6));

	export function push(): boolean {
		if (isFull) {
			return false;
		}
		operations.push(`TOS++ (${TOS} ---> ${TOS + 1})`);
		TOS++;
		const element = String.fromCharCode(65 + TOS);
		stack.push(element);
		operations.push(`Stack[${TOS}] = ${element}`);
		return true;
	}

	export function pop(): string | null {
		if (isEmpty) {
			return null;
		}
		const element = stack.pop();
		operations.push(`TOS-- (${TOS} ---> ${TOS - 1})`);
		TOS--;
		return element || null;
	}

	export function peek(): string | null {
		return stack[TOS] || null;
	}

	export function isStackEmpty(): boolean {
		return isEmpty;
	}
	export function isStackFull(): boolean {
		return isFull;
	}
	export function getTOS(): number {
		return TOS;
	}
	export function getStack() {
		return stack;
	}
	export function getOperations() {
		return operations;
	}
	export function getMax() {
		return MAX;
	}
	export function makeFull() {
		for (let i = 0; i < MAX; i++) {
			push();
		}
	}
	export function makeEmpty() {
		for (let i = 0; i < MAX; i++) {
			pop();
		}
	}
	export function getRemaining() {
		return MAX - stack.length;
	}
	export function reset() {
		TOS = -1;
		stack = [];
		operations = [];
	}
</script>

<div class="playground">
	<div class="playground-container">
		<!-- stack -->
		<div class={cn('stack-box', isFull && 'full', isEmpty && 'empty')}>
			{#each stack as item, i}
				{@const isHead = TOS === i}
				<!-- svelte-ignore a11y_click_events_have_key_events -->
				<!-- svelte-ignore a11y_no_static_element_interactions -->
				<div
					in:fly|local={{ duration: IN_OUT_ANIMATION_DURATION, y: -200 }}
					out:fly|local={{ duration: IN_OUT_ANIMATION_DURATION, y: -200 }}
					class={cn('stack-item', isHead && 'head')}
					onclick={() => isHead && pop()}
				>
					{item}
					<span class="index">
						{i}
					</span>
				</div>
			{/each}
		</div>
		<div class="stack-actions">
			<Button size="sm" disabled={isFull} onclick={push} class="gap-2"
				><ArrowBendLeftDown class="min-w-fit" /> Push</Button
			>
			<Button size="sm" disabled={isEmpty} onclick={pop} class="gap-2"
				><ArrowBendUpLeft class="min-w-fit" />Pop</Button
			>
		</div>
	</div>

	<div class="stack-info">
		<div class="core-info">
			<div class="flex gap-2">
				<span class="font-semibold">TOS:</span>
				<span>{TOS}</span>
			</div>
			<div class="flex gap-2">
				<span class="font-semibold">Peek:</span>
				<span>{peek() || 'NULL'}</span>
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
				{#key i}
					<div in:slide|local class={cn('operation', isLast && 'font-semibold')}>
						{operation}
						{#if isLast}
							<span class="text-xs italic text-muted-foreground ml-8">(Most Recent)</span>
						{/if}
					</div>
				{/key}
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

		@media only screen and (max-width: 1024px) {
			flex-direction: column;
			gap: theme('gap.4');
		}
	}

	.playground-container {
		display: flex;
		gap: theme('gap.4');

		@media only screen and (max-width: 1024px) {
			width: min-content;
			gap: theme('gap.2');
			flex-direction: column;
			margin: 0 auto;
		}
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

		overflow: hidden;

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
			background-color: theme('backgroundColor.green.100');
			border-color: theme('borderColor.green.600');

			&::after {
				content: 'Empty';
				position: absolute;
				top: 50%;
				left: 50%;
				transform: translate(-50%, -50%);
				font-size: theme('fontSize.sm');
				font-weight: theme('fontWeight.semibold');
				color: theme('colors.green.900');
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

		@media only screen and (max-width: 1024px) {
			width: 100%;
			display: grid;
			grid-template-columns: repeat(2, minmax(0, 1fr));
			gap: theme('gap.1');
		}
	}

	.stack-info {
		display: flex;
		flex-direction: column;

		gap: theme('gap.1');

		& .core-info {
			@apply grid-cols-2;
			display: grid;
			gap: theme('gap.1') theme('gap.16');
		}

		& .operations {
			& .operation {
				font-size: theme('fontSize.sm');
				color: theme('colors.foreground');
			}
		}
	}

	:global(.dark) .stack-box {
		&.empty {
			background-color: theme('backgroundColor.green.800');

			&::after {
				color: theme('colors.green.100');
			}
		}

		&.full {
			background-color: theme('colors.red.900');
		}
	}
</style>
