<script>
	import { page } from '$app/stores';
	import { CONST } from '$lib/const';
	import { ArrowLeft } from 'phosphor-svelte';
</script>

<div class="header">
	<!-- head -->
	<div class="head">
		<a href={CONST.ROUTES.HOME()}>
			{#if $page.url.pathname !== CONST.ROUTES.HOME()}
				<ArrowLeft />
			{/if}
			<span>{CONST.TITLE}</span>
		</a>
	</div>

	<!-- body -->
	<div class="body"></div>

	<!-- tail -->
	<div class="tail"></div>
</div>

<style lang="postcss">
	.header {
		width: 100%;
		height: fit-content;

		padding: theme('padding.4') theme('padding.6');

		display: flex;
		justify-content: center;
		align-items: center;
		gap: theme('gap.4');

		border-bottom: 1px solid theme('borderColor.border');

		& .head {
			font-size: theme('fontSize.2xl');
			font-weight: theme('fontWeight.bold');

			& a {
				display: flex;
				justify-content: center;
				align-items: center;
				gap: theme('gap.2');
			}
		}

		& .body {
			flex-grow: 1;
		}
	}
</style>
