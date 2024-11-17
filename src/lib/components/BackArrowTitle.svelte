<script lang="ts">
	import BackArrow from './BackArrow.svelte';
	import { Button } from '$lib/components/ui/button';
	import { Moon, Sun } from 'phosphor-svelte';
	import { toggleMode, mode } from 'mode-watcher';
	import ThemeSwitcher from './ThemeSwitcher.svelte';

	let { title, href }: { title: string; href?: string } = $props();
</script>

<div class="prose flex justify-between min-w-full">
	<h1 class="flex gap-4"><span><BackArrow {href} /></span>{title}</h1>
	<div>
		<!-- <ThemeSwitcher /> -->
	</div>
</div>
