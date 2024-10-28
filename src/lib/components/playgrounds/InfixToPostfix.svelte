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
	import StackBox from '$lib/components/shared/StackBox.svelte';
	import OperationsListBox from '$lib/components/shared/OperationsListBox.svelte';

	type Props = {
		infix?: string;
	};

	const IN_OUT_ANIMATION_DURATION = 300;
	const OPERATORS = ['^', '/', '*', '+', '-'];
	const RIGHT_ASSOCIATIVE_OPERATORS = ['^'];
	const SEPARATORS = [' '];

	let { infix = $bindable('(a + b) * c') }: Props = $props();

	type Step = {
		type: 'push' | 'pop' | 'pop_push';
		target: 'stack' | 'postfix';
		value: string;
		state: {
			highlightIndexes: number[];
			operatorStack: string[];
			postfix: string[];
		};
	};

	let infixInputValue = $state<string>(infix);
	let infixEditMode = $state(false);
	let operatorStack = $state<string[]>([]);
	let postfix = $state<string[]>([]);
	let stepIndex = $state(-1);
	let isPlaying = $state(false);
	let playInterval = $state<number | null>(null);
	let highlightIndexes = $state<number[]>([]);

	let steps = $derived<Step[]>(computeSteps(infix));
	let isFirstStep = $derived(stepIndex === -1);
	let isLastStep = $derived(stepIndex === steps.length - 1);

	$inspect('steps?', steps);
	$inspect('stepIndex?', stepIndex);

	export function setInfix(infixValue: string) {
		infix = infixValue;
		infixEditMode = false;
		operatorStack = [];
		postfix = [];
		stepIndex = -1;
		isPlaying = false;
		playInterval && clearInterval(playInterval);
		playInterval = null;
		highlightIndexes = [];
	}

	export function computeSteps(infixExpression: string) {
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

		function getState() {
			return {
				highlightIndexes: [...highlightIndexes],
				operatorStack: [...internalOperatorStack.map((char) => char.char)],
				postfix: [...internalPostfixStack.map((char) => char.char)]
			};
		}

		for (let i = 0; i < infixExpression.length; i++) {
			const char = infixExpression[i];

			// Ignore invalid characters
			if (/[a-zA-Z^*+-/\(\)\%]/.test(char) === false) continue;

			// Handle opening bracket
			if (isOpeningBracket(char)) {
				highlightIndexes.push(i);
				internalOperatorStack.push({ char, index: i });
				computedSteps.push({
					target: 'stack',
					type: 'push',
					value: char,
					state: getState()
				});
				highlightIndexes.pop();
				continue;
			}

			// Handle operands
			if (isOperand(char)) {
				highlightIndexes.push(i);
				internalPostfixStack.push({ char, index: i });
				computedSteps.push({
					target: 'postfix',
					type: 'push',
					value: char,
					state: getState()
				});
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
							target: 'stack',
							value: operator.char,
							state: getState()
						});
						highlightIndexes.pop();
					}
				}
				internalOperatorStack.push({
					char,
					index: i
				});
				computedSteps.push({
					type: 'push',
					target: 'stack',
					value: char,
					state: getState()
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
						internalPostfixStack.push(operator);
						computedSteps.push({
							type: 'pop_push',
							target: 'postfix',
							value: operator.char,
							state: getState()
						});
						highlightIndexes.pop();
					}
				}
				// Pop the opening bracket '(' from the stack
				internalOperatorStack.pop();
				computedSteps.push({
					type: 'pop',
					target: 'stack',
					value: '(',
					state: getState()
				});
				highlightIndexes.pop();
				continue;
			}
		}

		// Pop all remaining operators in the stack to postfix
		while (internalOperatorStack.length > 0) {
			const operator = internalOperatorStack[internalOperatorStack.length - 1];
			if (operator) {
				highlightIndexes.push(operator.index);
				internalOperatorStack.pop();
				internalPostfixStack.push(operator);
				computedSteps.push({
					type: 'pop_push',
					target: 'postfix',
					value: operator.char,
					state: getState()
				});
				highlightIndexes.pop();
			}
		}

		return computedSteps;
	}

	export function getOperatorPriority(operator: string) {
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

	export function enterInfixEditMode() {
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

	export function isOperand(char: string) {
		return /[a-zA-Z]/.test(char);
	}

	export function isOperator(char: string) {
		return OPERATORS.includes(char);
	}

	export function isOpeningBracket(char: string) {
		return char === '(';
	}

	export function isClosingBracket(char: string) {
		return char === ')';
	}

	export function isSeparator(char: string) {
		return SEPARATORS.includes(char);
	}

	export function isRightAssociative(char: string) {
		return RIGHT_ASSOCIATIVE_OPERATORS.includes(char);
	}

	export function moveStepBackward() {
		// get step and decrease stepIndex
		const step = steps[stepIndex - 1];
		stepIndex--;

		// set highlighted indexes coming from step
		highlightIndexes = step?.state.highlightIndexes || [];

		operatorStack = step?.state.operatorStack || [];
		postfix = step?.state.postfix || [];
	}

	export function moveStepForward() {
		// increase stepIndex and get step
		const step = steps[stepIndex + 1];
		stepIndex++;

		// set highlighted indexes coming from step
		highlightIndexes = step?.state.highlightIndexes || [];

		operatorStack = step?.state.operatorStack || [];
		postfix = step?.state.postfix || [];

		if (isLastStep) {
			highlightIndexes = [];
		}
	}

	export function play() {
		playInterval = setInterval(() => {
			moveStepForward();
		}, 1000);
		isPlaying = true;
	}

	export function pause() {
		if (!isPlaying || !playInterval) {
			return;
		}
		clearInterval(playInterval);
		isPlaying = false;
	}

	export function resetPlay() {
		pause();
		// a neat trick to reset the whole playground
		setInfix(infix);
	}

	export function getOperationFromStep(step: Step) {
		return step.type === 'push'
			? `Push ${step.value} onto ${step.target}`
			: step.type === 'pop_push'
				? `Pop ${step.value} from ${step.target === 'postfix' ? 'stack' : 'postfix'} to ${step.target}`
				: `Pop ${step.value} from ${step.target}`;
	}

	export function skipAllSteps() {
		// go through all remaining steps (n) and move n times forward
		for (let i = stepIndex; i < steps.length - 1; i++) {
			moveStepForward();
		}
	}

	function handleInfixInput() {
		if (infixInputValue.trim() === '') {
			return;
		}
		setInfix(infixInputValue);
		skipAllSteps();
	}

	onMount(() => {
		skipAllSteps();
	});
</script>

{#snippet ExpressionCharacterDisplayItem(char: string, options: { className?: string } = {})}
	{@const baseClass = cn('text-xl lg:text-2xl font-semibold', options.className)}
	{#if isOperand(char)}
		<div class={cn(baseClass, 'text-foreground')}>{char}</div>
	{:else if isOperator(char)}
		<div class={cn(baseClass, 'text-red-600 dark:text-red-400')}>{char}</div>
	{:else if isOpeningBracket(char)}
		<div class={cn(baseClass, 'text-green-600 dark:text-green-400')}>{char}</div>
	{:else if isClosingBracket(char)}
		<div class={cn(baseClass, 'text-green-600 dark:text-green-400')}>{char}</div>
	{/if}
{/snippet}

{#snippet InfixExpressionEditor()}
	<input
		bind:value={infixInputValue}
		placeholder="Enter Infix Expression"
		class="w-full outline-none p-0 text-xl lg:text-2xl font-semibold border-b tracking-widest"
		onkeydown={(event) => {
			event.key === 'Enter' && handleInfixInput();
			if (event.key === 'Escape') {
				infixEditMode = false;
			}
		}}
	/>
{/snippet}

{#snippet InfixDisplay(infix: string)}
	<div class="flex flex-grow gap-0.5">
		{#each infix.split('') as char, i}
			{@render ExpressionCharacterDisplayItem(char, {
				className: cn(
					'p-0.5 transition',
					highlightIndexes.includes(i) && 'bg-yellow-300 dark:bg-yellow-900 -translate-y-1'
				)
			})}
		{/each}
	</div>
{/snippet}

{#snippet InfixExpressionSection()}
	<div class="h-16 flex flex-col justify-center">
		<div class="text-xs lg:text-sm font-semibold">Infix Expression</div>
		<div class="flex w-full items-end gap-2">
			<!-- svelte-ignore a11y_no_static_element_interactions -->
			{#if infixEditMode}
				{@render InfixExpressionEditor()}
			{:else}
				{@render InfixDisplay(infix)}
			{/if}
			<Button
				variant={infixEditMode ? 'default' : 'secondary'}
				size="sm"
				onclick={infixEditMode ? handleInfixInput : enterInfixEditMode}
			>
				{#if infixEditMode}
					<Check class="text-lg lg:text-xl" />
				{:else}
					<PencilSimple class="text-lg lg:text-xl" />
				{/if}
			</Button>
		</div>
	</div>
{/snippet}

{#snippet StackBoxContainer()}
	<div class="lg:contents flex gap-8">
		<div class="w-fit flex justify-center items-center flex-col">
			<p class="m-1 text-center font-semibold text-sm">Operator Stack</p>
			<!-- stack box -->
			<StackBox bind:stack={operatorStack} />
		</div>
		<div class="contents w-full lg:hidden">
			{@render ActionsContainer()}
		</div>
	</div>
{/snippet}

{#snippet PostfixExpressionDisplay(postfix: string[])}
	<div class="flex flex-col gap-2 min-h-fit">
		<div class="font-semibold text-xs lg:text-sm">Postfix Expression</div>
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
{/snippet}

{#snippet StepProgressCounter()}
	<div class="text-center font-semibold text-sm">
		Steps: {stepIndex + 1}/{steps.length}
	</div>
{/snippet}

{#snippet BackForwardActionsContainer()}
	<div class="grid grid-cols-2 gap-1">
		<Button size="sm" disabled={isFirstStep} onclick={moveStepBackward}
			><CaretLeft size={18} /></Button
		>
		<Button size="sm" disabled={isLastStep} onclick={moveStepForward}
			><CaretRight size={18} /></Button
		>
	</div>
{/snippet}

{#snippet SingletonPlayPauseResetActionsButton()}
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
{/snippet}

{#snippet SingletonResetActionButton()}
	{#if stepIndex !== -1 && stepIndex < steps.length - 1}
		<Button class="gap-1" onclick={resetPlay}>
			<ArrowClockwise size={18} />
			Reset
		</Button>
	{/if}
{/snippet}

{#snippet SingletonSkipActionButton()}
	{#if stepIndex !== steps.length - 1}
		<Button class="gap-1" onclick={skipAllSteps}>
			<CaretDoubleRight size={18} />
			Skip
		</Button>
	{/if}
{/snippet}

{#snippet ActionsContainer()}
	<div class="w-full lg:w-fit flex flex-col gap-2">
		{@render StepProgressCounter()}
		<!-- back-forward -->
		{@render BackForwardActionsContainer()}
		<!-- play -->
		<div class="grid lg:grid-cols-1 grid-cols-2 gap-1">
			{@render SingletonPlayPauseResetActionsButton()}
			{@render SingletonSkipActionButton()}
			{@render SingletonResetActionButton()}
		</div>
	</div>
{/snippet}

<div class="playground">
	{@render InfixExpressionSection()}

	<div class="flex gap-4 flex-col lg:flex-row">
		<!-- stack container -->
		{@render StackBoxContainer()}

		<div class="flex-grow flex flex-col gap-1 lg:gap-3">
			<!-- postfix expression -->
			{@render PostfixExpressionDisplay(postfix)}

			<OperationsListBox
				class="max-h-44"
				operations={steps.slice(0, stepIndex + 1).map(getOperationFromStep)}
			/>
		</div>
		<!-- actions -->
		<div class="hidden lg:contents">
			{@render ActionsContainer()}
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
