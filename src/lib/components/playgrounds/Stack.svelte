<script lang="ts">
	import { Button } from '$lib/components/ui/button';
	import { ArrowBendLeftDown, ArrowBendUpLeft } from 'phosphor-svelte';
	import StackBox from '$lib/components/shared/StackBox.svelte';
	import OperationsListBox from '$lib/components/shared/OperationsListBox.svelte';

	const MAX = 5;
	const IN_OUT_ANIMATION_DURATION = 300;

	type Props = {};

	const {}: Props = $props();

	let stack = $state<string[]>([]);
	let TOS: number = $state(-1);
	const isEmpty = $derived(TOS === -1);
	const isFull = $derived(stack.length >= MAX);

	let operations = $state<string[]>([]);

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
		<StackBox bind:stack capacity={MAX} />
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

		<OperationsListBox {operations} class="max-h-36" />
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

		gap: theme('gap.4');

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

		@media only screen and (max-width: 1024px) {
			gap: theme('gap.2');
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
