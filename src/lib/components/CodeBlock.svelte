<script lang="ts">
	import { codeToHtml, type BundledLanguage } from 'shiki';

	type Props = {
		code: string;
		lang?: BundledLanguage;
	};

	const { code, lang = 'c' }: Props = $props();

	let html: string = $state('');

	function indent(text: string) {
		if (!/\t/.test(text.trim())) {
			return text;
		}

		const code = text.trim().split('\n');

		if (code.length === 1) {
			return text.trim();
		}

		const tabs = code
			.map((line) => line.split('').filter((char) => char === '\t'))
			.filter((line) => line.length !== 0)
			.sort((a, b) => a.length - b.length)[0]
			.join('');

		if (tabs === '\t') {
			return text.trim();
		}

		return code.map((line) => line.replace(tabs, '')).join('\n');
	}

	$effect(() => {
		(async () => {
			html = await codeToHtml(indent(code), { lang, theme: 'github-dark' });
		})();
	});
</script>

{@html html}
