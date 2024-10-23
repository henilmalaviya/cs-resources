<script>
	import BackArrow from './BackArrow.svelte';

	let { title = '' } = $props();
</script>

<div class="prose">
	<h1 class="flex gap-4"><span><BackArrow /></span>{title}</h1>
</div>
