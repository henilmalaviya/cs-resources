<script lang="ts">
	import BackArrow from './BackArrow.svelte';

	let { title, href }: { title: string; href?: string } = $props();
</script>

<div class="prose">
	<h1 class="flex gap-4"><span><BackArrow {href} /></span>{title}</h1>
</div>
