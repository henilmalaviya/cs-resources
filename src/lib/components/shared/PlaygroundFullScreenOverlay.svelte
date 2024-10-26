<script lang="ts">
	import type { Snippet } from 'svelte';

	type Props = {
		children: Snippet;
	};

	let { children }: Props = $props();
</script>

<div class="fixed inset-0 bg-background z-10 flex justify-center w-full overflow-auto">
	<div class="py-6 px-4 flex flex-col gap-4 w-full prose">
		{@render children()}
	</div>
</div>
