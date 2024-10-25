<script lang="ts">
	import { wrapperWidth } from '$lib/store/global';
	import '$lib/styles/global.css';
	import { onMount } from 'svelte';

	let { children } = $props();

	let wrapperDiv: HTMLDivElement;

	onMount(() => {
		wrapperWidth.set(wrapperDiv.clientWidth);
	});
</script>

<div bind:this={wrapperDiv} class="wrapper">
	{@render children()}
</div>

<style lang="postcss">
	.wrapper {
		width: 44rem;
		height: 100%;
		margin: 0 auto;
		padding: theme('padding.16') theme('padding.8');

		@media (max-width: 768px) {
			margin: 0;
			width: 100%;
			padding: theme('padding.16') theme('padding.4');
		}
	}
</style>
