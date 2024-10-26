<script lang="ts">
	import { cn } from '$lib/utils/shadcn';

	type Props = {
		operations: string[];
		ref?: HTMLDivElement;
		class?: string;
		autoScroll?: boolean;
	};

	let {
		operations = $bindable(),
		ref = $bindable(),
		class: className = $bindable(''),
		autoScroll = $bindable(true)
	}: Props = $props();

	export function scrollToBottom() {
		if (ref) {
			ref.scrollTop = ref.scrollHeight;
		}
	}

	$effect(() => {
		if (autoScroll && operations.length > 0) {
			scrollToBottom();
		}
	});
</script>

<div>
	<h4 class="my-1">Operations</h4>
	<hr class="mb-4" />
	<div bind:this={ref} class={cn('operations-list fancy-scrollbar', className)}>
		{#if operations.length > 0}
			{#each operations as operation, index}
				{@const isLast = index === operations.length - 1}
				<div class={cn('text-sm text-foreground', isLast && 'font-semibold')}>
					{operation}
					{#if isLast}
						<span class="text-xs italic text-muted-foreground ml-8">(Most Recent)</span>
					{/if}
				</div>
			{/each}
		{:else}
			<span class="text-muted-foreground italic text-sm">No operations yet.</span>
		{/if}
	</div>
</div>

<style lang="postcss">
	.operations-list {
		width: 100%;

		display: flex;
		flex-direction: column;
		gap: theme('gap[0.5]');

		overflow-y: auto;
	}
</style>
