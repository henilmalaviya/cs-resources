<script lang="ts">
	import BackArrowTitle from '$lib/components/BackArrowTitle.svelte';
	import { CONST } from '$lib/const';
</script>

{#snippet LinkItem(label: string, href: string)}
	<div>
		<a {href}>{label}</a>
	</div>
{/snippet}

<div class="prose item-list">
	<BackArrowTitle title="Data Structures" />

	<div class="my-4"></div>

	{@render LinkItem('Stack', CONST.ROUTES.DS().STACK())}
</div>

<style lang="postcss">
	.item-list {
		display: flex;
		flex-direction: column;

		& a {
			text-decoration: underline;
			font-size: theme('fontSize.lg');
		}
	}
</style>
