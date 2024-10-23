<script lang="ts">
	import '$lib/styles/global.css';

	let { children } = $props();
</script>

<div class="wrapper">
	{@render children()}
</div>

<style lang="postcss">
	.wrapper {
		width: 42rem;
		height: 100%;
		margin: 0 auto;
		padding: theme('padding.16') 0;
	}
</style>
