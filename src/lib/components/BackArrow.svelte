<script lang="ts">
	import { goto } from '$app/navigation';
	import { ArrowLeft } from 'phosphor-svelte';

	let { href }: { href?: string } = $props();
</script>

<ArrowLeft class="cursor-pointer" onclick={() => (href ? goto(href) : window.history.back())} />
