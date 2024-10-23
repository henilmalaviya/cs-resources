<script lang="ts">
	import { CONST } from '$lib/const';
</script>

{#snippet LinkItem(label: string, href: string)}
	<div>
		<a {href}>{label}</a>
	</div>
{/snippet}

<div class="prose course-list">
	<h1>Courses</h1>

	{@render LinkItem('Data Structures', CONST.ROUTES.DS()._())}
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
