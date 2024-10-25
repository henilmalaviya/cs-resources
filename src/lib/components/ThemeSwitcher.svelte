<script>
	import { mode, toggleMode } from 'mode-watcher';

	import { Button } from '$lib/components/ui/button';
	import { Moon, Sun } from 'phosphor-svelte';
</script>

<Button onclick={toggleMode} size="sm" variant="ghost">
	{#if $mode === 'dark'}
		<Sun class="min-w-fit" size={20} />
	{:else}
		<Moon class="min-w-fit" size={20} />
	{/if}
</Button>
