<script lang="ts">
	import { cn } from '$lib/utils/shadcn';
	import { fly } from 'svelte/transition';

	type Props = {
		stack?: string[];
		animationDuration?: number;
		class?: string;
		capacity?: number;
	};

	let {
		stack = $bindable([]),
		animationDuration = $bindable(300),
		class: className = $bindable(''),
		capacity = $bindable(-1)
	}: Props = $props();

	const isFull = $derived(capacity === -1 ? false : stack.length >= capacity);
	const isEmpty = $derived(stack.length === 0);
</script>

<div class={cn('stack-box', isFull && 'full', isEmpty && 'empty', className)}>
	{#each stack as item, i}
		{@const isHead = i === stack.length - 1}
		<div
			in:fly|local={{ duration: animationDuration, y: -200 }}
			out:fly|local={{ duration: animationDuration, y: -200 }}
			class={cn('stack-item', isHead && '')}
		>
			{item}
			<span class="index">
				{i}
			</span>
		</div>
	{/each}
</div>

<style lang="postcss">
	.stack-box {
		width: theme('width.40');
		min-height: theme('height.72');

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

			padding: theme('padding.2');
			position: relative;

			display: flex;
			justify-content: center;
			align-items: center;

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
