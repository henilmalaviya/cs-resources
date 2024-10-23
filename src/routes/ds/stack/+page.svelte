<script>
	import BackArrowTitle from '$lib/components/BackArrowTitle.svelte';
	import StackPlayground from '$lib/components/playgrounds/Stack.svelte';
	import { CONST } from '$lib/const';
</script>

<div class="prose min-w-full">
	<BackArrowTitle href={CONST.ROUTES.DS()._()} title="Stack" />

	<div class="my-4"></div>

	<div class="px-4 py-4 border-2 border-dashed flex flex-col gap-4">
		<h3 class="m-0">Playground</h3>
		<hr class="m-0 mb-2" />

		<StackPlayground />
	</div>
</div>
