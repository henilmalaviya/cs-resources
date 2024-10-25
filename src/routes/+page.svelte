<script lang="ts">
	import PageUnderDevelopment from '$lib/components/PageUnderDevelopment.svelte';
	import ThemeSwitcher from '$lib/components/ThemeSwitcher.svelte';
	import { CONST } from '$lib/const';
</script>

{#snippet LinkItem(label: string, href: string)}
	<div>
		<a {href}>{label}</a>
	</div>
{/snippet}

<div class="prose course-list">
	<div class="w-full flex justify-between">
		<h1>Courses</h1>
		<div>
			<ThemeSwitcher />
		</div>
	</div>

	<PageUnderDevelopment />

	<ul>
		<li>
			{@render LinkItem('Data Structures', CONST.ROUTES.DS()._())}
		</li>
	</ul>
</div>

<style lang="postcss">
	.course-list {
		display: flex;
		flex-direction: column;
		min-width: 100%;

		& a {
			text-decoration: underline;
			font-size: theme('fontSize.lg');
		}
	}
</style>
