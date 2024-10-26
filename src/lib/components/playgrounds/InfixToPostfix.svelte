<script lang="ts">
	import { Button } from '$lib/components/ui/button';
	import { cn } from '$lib/utils/shadcn';
	import {
		Check,
		PencilSimple,
		CaretLeft,
		CaretRight,
		Play,
		Pause,
		ArrowClockwise,
		CaretDoubleRight
	} from 'phosphor-svelte';
	import { onMount } from 'svelte';
	import { fly } from 'svelte/transition';

	type Props = {
		infix?: string;
	};

	const IN_OUT_ANIMATION_DURATION = 300;
	const OPERATORS = ['^', '/', '*', '+', '-'];
	const RIGHT_ASSOCIATIVE_OPERATORS = ['^'];

	let { infix = $bindable('a + (b * c) / d - e ^ f * (g + h)') }: Props = $props();

	type Step = {
		type: 'push' | 'pop';
		to: 'stack' | 'postfix';
		value: string;
		highlightIndexes: number[];
	};

	let infixInputValue = $state<string>(infix);
	let infixEditMode = $state(false);
	let operatorStack = $state<string[]>([]);
	let postfix = $state<string[]>([]);
	let steps = $derived<Step[]>(computeSteps(infix));
	let stepIndex = $state(-1);
	let isFirstStep = $derived(stepIndex === -1);
	let isLastStep = $derived(stepIndex === steps.length - 1);
	let isPlaying = $state(false);
	let playInterval = $state<number | null>(null);
	let highlightIndexes = $state<number[]>([]);
	let operationsRef = $state<HTMLDivElement | null>(null);

	function handleInfixInput() {
		if (infixInputValue.trim() === '') {
			return;
		}
		infix = infixInputValue;
		infixInputValue = '';
		infixEditMode = false;
		stepIndex = -1;
		operatorStack = [];
		postfix = [];
		isPlaying = false;
		playInterval && clearInterval(playInterval);
		playInterval = null;
		highlightIndexes = [];

		skipSteps();
	}

	function computeSteps(infixExpression: string) {
		let computedSteps: Step[] = [];
		let internalOperatorStack: {
			char: string;
			index: number;
		}[] = [];
		let internalPostfixStack: {
			char: string;
			index: number;
		}[] = [];
		let highlightIndexes: number[] = [];

		for (let i = 0; i < infixExpression.length; i++) {
			const char = infixExpression[i];

			// Ignore invalid characters
			if (/[a-zA-Z^*+-/\(\)\%]/.test(char) === false) continue;

			// Handle opening bracket
			if (isOpeningBracket(char)) {
				highlightIndexes.push(i);
				computedSteps.push({
					type: 'push',
					to: 'stack',
					value: char,
					highlightIndexes: [...highlightIndexes]
				});
				internalOperatorStack.push({ char, index: i });
				highlightIndexes.pop();
				continue;
			}

			// Handle operands
			if (isOperand(char)) {
				highlightIndexes.push(i);
				computedSteps.push({
					type: 'push',
					to: 'postfix',
					value: char,
					highlightIndexes: [...highlightIndexes]
				});
				internalPostfixStack.push({ char, index: i });
				highlightIndexes.pop();
				continue;
			}

			// Handle operators
			if (isOperator(char)) {
				highlightIndexes.push(i);
				while (
					internalOperatorStack.length > 0 &&
					internalOperatorStack[internalOperatorStack.length - 1].char !== '(' &&
					(getOperatorPriority(internalOperatorStack[internalOperatorStack.length - 1].char) >
						getOperatorPriority(char) ||
						(getOperatorPriority(internalOperatorStack[internalOperatorStack.length - 1].char) ===
							getOperatorPriority(char) &&
							!isRightAssociative(char)))
				) {
					const operator = internalOperatorStack.pop();
					if (operator) {
						highlightIndexes.push(operator.index);
						computedSteps.push({
							type: 'pop',
							to: 'stack',
							value: operator.char,
							highlightIndexes: [...highlightIndexes]
						});
						internalPostfixStack.push(operator);
						computedSteps.push({
							type: 'push',
							to: 'postfix',
							value: operator.char,
							highlightIndexes: [...highlightIndexes]
						});
						highlightIndexes.pop();
					}
				}
				computedSteps.push({
					type: 'push',
					to: 'stack',
					value: char,
					highlightIndexes: [...highlightIndexes]
				});
				internalOperatorStack.push({
					char,
					index: i
				});
				highlightIndexes.pop();
				continue;
			}

			// Handle closing bracket
			if (isClosingBracket(char)) {
				highlightIndexes.push(i);
				while (
					internalOperatorStack.length > 0 &&
					internalOperatorStack[internalOperatorStack.length - 1].char !== '('
				) {
					const operator = internalOperatorStack.pop();
					if (operator) {
						highlightIndexes.push(operator.index);
						computedSteps.push({
							type: 'pop',
							to: 'stack',
							value: operator.char,
							highlightIndexes: [...highlightIndexes]
						});
						internalPostfixStack.push(operator);
						computedSteps.push({
							type: 'push',
							to: 'postfix',
							value: operator.char,
							highlightIndexes: [...highlightIndexes]
						});
						highlightIndexes.pop();
					}
				}
				// Pop the opening bracket '(' from the stack
				computedSteps.push({
					type: 'pop',
					to: 'stack',
					value: char,
					highlightIndexes: [...highlightIndexes]
				});
				highlightIndexes.pop();
				internalOperatorStack.pop();
				continue;
			}
		}

		// Pop all remaining operators in the stack to postfix
		while (internalOperatorStack.length > 0) {
			const operator = internalOperatorStack.pop();
			if (operator) {
				highlightIndexes.push(operator.index);
				computedSteps.push({
					type: 'pop',
					to: 'stack',
					value: operator.char,
					highlightIndexes: [...highlightIndexes]
				});
				internalPostfixStack.push(operator);
				computedSteps.push({
					type: 'push',
					to: 'postfix',
					value: operator.char,
					highlightIndexes: [...highlightIndexes]
				});
				highlightIndexes.pop();
			}
		}

		return computedSteps;
	}

	function getOperatorPriority(operator: string) {
		switch (operator) {
			case '^':
				return 3;
			case '*':
				return 2;
			case '/':
				return 2;
			case '+':
				return 1;
			case '-':
				return 1;
			default:
				return 0;
		}
	}

	function enterInfixEditMode() {
		infixEditMode = true;
		infixInputValue = infix;

		function escapeChecker(event: KeyboardEvent) {
			if (event.key === 'Escape') {
				infixEditMode = false;
				document.body.removeEventListener('keydown', escapeChecker);
			}
		}
		document.body.addEventListener('keydown', escapeChecker);
	}

	function isOperand(char: string) {
		return /[a-zA-Z]/.test(char);
	}

	function isOperator(char: string) {
		return OPERATORS.includes(char);
	}

	function isOpeningBracket(char: string) {
		return char === '(';
	}

	function isClosingBracket(char: string) {
		return char === ')';
	}

	function isSeparator(char: string) {
		return char === ' ';
	}

	function isRightAssociative(char: string) {
		return RIGHT_ASSOCIATIVE_OPERATORS.includes(char);
	}

	function moveStepBackward() {
		if (stepIndex === -1) {
			return;
		}

		const step = steps[stepIndex--];
		highlightIndexes = step.highlightIndexes;

		if (!step) return;

		if (step.to === 'stack' && step.type === 'push') {
			operatorStack.pop();
		} else if (step.to === 'stack' && step.type === 'pop') {
			operatorStack.push(step.value);
		}

		if (step.to === 'postfix' && step.type === 'push') {
			postfix.pop();
		} else if (step.to === 'postfix' && step.type === 'pop') {
			postfix.push(step.value);
		}

		if (stepIndex === -1) {
			highlightIndexes = [];
		}

		requestAnimationFrame(() => {
			if (operationsRef) {
				operationsRef.scrollTop = operationsRef.scrollHeight;
			}
		});
	}

	function moveStepForward() {
		if (stepIndex === steps.length - 1) {
			return;
		}

		const step = steps[++stepIndex];

		highlightIndexes = step.highlightIndexes;

		if (step.type === 'push' && step.to === 'stack') {
			operatorStack.push(step.value);
		} else if (step.type === 'pop' && step.to === 'stack') {
			operatorStack.pop();
		}

		if (step.to === 'postfix' && step.type === 'push') {
			postfix.push(step.value);
		} else if (step.to === 'postfix' && step.type === 'pop') {
			postfix.pop();
		}

		if (stepIndex === steps.length - 1) {
			highlightIndexes = [];
		}

		requestAnimationFrame(() => {
			if (operationsRef) {
				operationsRef.scrollTop = operationsRef.scrollHeight;
			}
		});
	}

	function play() {
		playInterval = setInterval(() => {
			moveStepForward();
		}, 1000);
		isPlaying = true;
	}

	function pause() {
		if (!isPlaying || !playInterval) {
			return;
		}
		clearInterval(playInterval);
		isPlaying = false;
	}

	function resetPlay() {
		pause();
		stepIndex = -1;
		highlightIndexes = [];
		operatorStack = [];
		postfix = [];
	}

	function getOperationFromStep(step: Step) {
		return step.type === 'push'
			? `Push ${step.value} onto ${step.to}`
			: `Pop ${step.value} from ${step.to}`;
	}

	function skipSteps() {
		for (let i = stepIndex; i < steps.length - 1; i++) {
			moveStepForward();
		}
	}

	onMount(() => {
		skipSteps();
	});
</script>

{#snippet ExpressionCharacterDisplayItem(char: string)}
	{@const baseClass = 'text-2xl font-semibold'}
	{#if isOperand(char)}
		<div class={cn(baseClass, 'text-foreground')}>{char}</div>
	{:else if isOperator(char)}
		<div class={cn(baseClass, 'text-red-600 dark:text-red-400')}>{char}</div>
	{:else if isOpeningBracket(char)}
		<div class={cn(baseClass, 'text-green-600 dark:text-green-400')}>{char}</div>
	{:else if isClosingBracket(char)}
		<div class={cn(baseClass, 'text-green-600 dark:text-green-400')}>{char}</div>
	{:else if isSeparator(char)}
		<!--  -->
	{/if}
{/snippet}

<div class="playground max-h-40">
	<div class="h-20 flex flex-col justify-center">
		<div class="text-sm font-semibold">Infix Expression</div>
		<div class="flex w-full items-end gap-2">
			<!-- svelte-ignore a11y_no_static_element_interactions -->
			{#if infixEditMode}
				<input
					bind:value={infixInputValue}
					placeholder="Enter Infix Expression"
					class="w-full outline-none p-2 text-2xl font-semibold border-b"
					onkeydown={(event) => {
						event.key === 'Enter' && handleInfixInput();
						if (event.key === 'Escape') {
							infixEditMode = false;
						}
					}}
				/>
			{:else}
				<div class="flex flex-grow gap-0.5">
					{#each infix.split('') as char, i}
						<div
							class={cn(
								'p-0.5 transition',
								highlightIndexes.includes(i) && 'bg-yellow-300 -translate-y-2'
							)}
						>
							{@render ExpressionCharacterDisplayItem(char)}
						</div>
					{/each}
				</div>
			{/if}
			<Button size="sm" onclick={infixEditMode ? handleInfixInput : enterInfixEditMode}>
				{#if infixEditMode}
					<Check size={20} />
				{:else}
					<PencilSimple size={20} />
				{/if}
			</Button>
		</div>
	</div>
	<!-- expression -->
	<div class="flex gap-4">
		<!-- stack container -->
		<div class="">
			<p class="m-1 text-center font-semibold text-sm">Operator Stack</p>
			<!-- stack box -->
			<div
				class={cn(
					'flex w-40 min-h-72 rounded-t-none flex-col-reverse gap-1 p-1 relative rounded-md border border-t-0 border-zinc-500',
					operatorStack.length === 0 &&
						'border-green-500 dark:bg-green-800 dark:after:text-white bg-green-100 after:absolute after:content-["Empty"] after:top-1/2 after:left-1/2 after:-translate-x-1/2 after:-translate-y-1/2 after:text-sm after:font-semibold'
				)}
			>
				{#each operatorStack as operator, i}
					<div
						class="w-full h-12 bg-foreground rounded text-background text-center p-2 flex justify-center items-center text-lg"
						in:fly|local={{ duration: IN_OUT_ANIMATION_DURATION, y: -200 }}
						out:fly|local={{ duration: IN_OUT_ANIMATION_DURATION, y: -200 }}
					>
						{operator}
					</div>
				{/each}
			</div>
		</div>
		<!-- operations -->
		<div class="flex-grow flex flex-col">
			<!-- postfix expression -->
			<div class="flex flex-col gap-2 min-h-fit">
				<div class="font-semibold text-sm">Postfix Expression</div>
				<div class="flex gap-1 min-h-10">
					{#if stepIndex === -1}
						<div class="flex justify-center items-center italic text-muted-foreground">
							Move a step forward...
						</div>
					{/if}
					{#each postfix as char}
						<div
							in:fly|local={{ duration: IN_OUT_ANIMATION_DURATION, x: 50 }}
							out:fly|local={{ duration: IN_OUT_ANIMATION_DURATION, x: 50 }}
						>
							{@render ExpressionCharacterDisplayItem(char)}
						</div>
					{/each}
				</div>
			</div>

			<h4>Operations</h4>
			<hr class="mb-4" />
			<div
				bind:this={operationsRef}
				class="flex flex-col gap-0.5 overflow-y-auto max-h-44 fancy-scrollbar"
			>
				{#if stepIndex > -1}
					{#each steps.slice(0, stepIndex + 1) as step, index}
						{@const isLast = index === stepIndex}
						<div class={cn('text-sm text-foreground', isLast && 'font-semibold')}>
							{getOperationFromStep(step)}
							{#if isLast}
								<span class="text-xs italic text-muted-foreground ml-8">(Most Recent)</span>
							{/if}
						</div>
					{/each}
				{:else}
					<span class="text-muted-foreground italic text-sm">No operations yet.</span>
				{/if}
			</div>
		</div>
		<!-- actions -->
		<div class="flex flex-col gap-2">
			<div class="text-center font-semibold text-sm">
				{stepIndex + 1}/{steps.length}
			</div>
			<!-- back-forward -->
			<div class="flex gap-1">
				<Button size="sm" disabled={isFirstStep} onclick={moveStepBackward}
					><CaretLeft size={18} /></Button
				>
				<Button size="sm" disabled={isLastStep} onclick={moveStepForward}
					><CaretRight size={18} /></Button
				>
			</div>
			<!-- play -->
			{#if isPlaying && stepIndex < steps.length - 1}
				<Button class="gap-1" onclick={pause}>
					<Pause size={18} />
					Pause
				</Button>
			{:else if stepIndex === steps.length - 1}
				<Button class="gap-1" onclick={resetPlay}>
					<ArrowClockwise size={18} />
					Reset
				</Button>
			{:else}
				<Button class="gap-1" onclick={play}>
					<Play size={18} />
					Run
				</Button>
			{/if}
			{#if stepIndex !== -1 && stepIndex < steps.length - 1}
				<Button class="gap-1" onclick={resetPlay}>
					<ArrowClockwise size={18} />
					Reset
				</Button>
			{/if}
			{#if stepIndex !== steps.length - 1}
				<Button class="gap-1" onclick={skipSteps}>
					<CaretDoubleRight size={18} />
					Skip
				</Button>
			{/if}
		</div>
	</div>
</div>

<style lang="postcss">
	.playground {
		width: 100%;
		min-height: fit-content;

		display: flex;
		flex-direction: column;
		gap: theme('gap.4');
	}

	.playground .infix-input-container {
		width: 100%;
		height: 100%;

		padding: theme('padding.16') theme('padding.8');
		display: flex;
		flex-direction: column;
		justify-content: center;
		gap: theme('gap.2');
	}
</style>
