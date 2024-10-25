<script lang="ts">
	import BackArrowTitle from '$lib/components/BackArrowTitle.svelte';
	import PageUnderDevelopment from '$lib/components/PageUnderDevelopment.svelte';
	import { CONST } from '$lib/const';
</script>

{#snippet LinkItem(label: string, href: string)}
	<div>
		<a {href}>{label}</a>
	</div>
{/snippet}

<div class="prose item-list">
	<BackArrowTitle href={CONST.ROUTES.HOME()} title="Data Structures" />

	<div class="my-4"></div>
	<PageUnderDevelopment />

	<ul>
		<li>
			{@render LinkItem('Stack', CONST.ROUTES.DS().STACK())}
		</li>
	</ul>
</div>

<style lang="postcss">
	.item-list {
		min-width: 100%;
		display: flex;
		flex-direction: column;

		& a {
			text-decoration: underline;
			font-size: theme('fontSize.lg');
		}
	}
</style>
