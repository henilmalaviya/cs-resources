<script lang="ts">
	import BackArrowTitle from '$lib/components/BackArrowTitle.svelte';
	import PageUnderDevelopment from '$lib/components/PageUnderDevelopment.svelte';
	import { CONST } from '$lib/const';
	import StackDisplaySvg from '$lib/assets/svg/stack.svg';
	import PostfixDisplaySvg from '$lib/assets/svg/postfix.svg';
	import { cn } from '$lib/utils/shadcn';

	function getItemClassName(color: string) {
		return `bg-${color}-50`;
	}
</script>

{#snippet Item({
	href,
	image,
	label,
	className
}: {
	image: string;
	label: string;
	href: string;
	className?: string;
})}
	<a
		{href}
		class={cn(
			'p-3 !no-underline transition hover:-translate-y-2 rounded shadow flex flex-col items-center gap-2',
			className
		)}
	>
		<img class="w-20 lg:w-24 h-full m-0" src={image} alt={label} />
		<span>{label}</span>
	</a>
{/snippet}

<div class="prose item-list">
	<BackArrowTitle href={CONST.ROUTES.HOME()} title="Data Structures" />

	<PageUnderDevelopment />

	<div class="my-4"></div>

	<div class="grid grid-cols-2 lg:grid-cols-3 gap-4">
		{@render Item({
			image: StackDisplaySvg,
			label: 'Stack',
			href: CONST.ROUTES.DS().STACK(),
			className:
				'bg-cyan-50 hover:bg-cyan-100 border border-cyan-200 hover:border-cyan-500 hover:shadow-cyan-300'
		})}
		{@render Item({
			image: PostfixDisplaySvg,
			label: 'Infix to Postfix',
			href: CONST.ROUTES.DS().INFIX_TO_POSTFIX(),
			className:
				'bg-red-50 hover:bg-red-100 border border-red-200 hover:border-red-500 hover:shadow-red-300'
		})}
	</div>
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
