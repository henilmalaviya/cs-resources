<script lang="ts">
	import BackArrowTitle from '$lib/components/BackArrowTitle.svelte';
	import CodeBlock from '$lib/components/CodeBlock.svelte';
	import StackPlayground from '$lib/components/playgrounds/Stack.svelte';
	import StackFunctionCallStackExampleSvg from '$lib/assets/svg/stack-function-call-stack-example.svg';
	import StackFunctionCallStackExampleDarkSvg from '$lib/assets/svg/stack-function-call-stack-example-dark.svg';
	import { CONST } from '$lib/const';
	import { Button } from '$lib/components/ui/button';
	import { ArrowClockwise, ArrowsOutSimple, ArrowsInSimple } from 'phosphor-svelte';
	import PageUnderDevelopment from '$lib/components/PageUnderDevelopment.svelte';
	import { cn } from '$lib/utils/shadcn';
	import { mode } from 'mode-watcher';

	let stackPlaygroundRef: ReturnType<typeof StackPlayground> | undefined = $state();

	let isPlaygroundFullScreen = $state(false);
	let scrollPosition = $state(0);
	let functionCallStackExampleImage = $state(StackFunctionCallStackExampleSvg);

	function togglePlaygroundFullScreen() {
		if (!isPlaygroundFullScreen) {
			// entering
			scrollPosition = window.scrollY;
		} else {
			// exiting
			setTimeout(() => {
				window.scrollTo(0, scrollPosition);
			}, 0);
		}

		isPlaygroundFullScreen = !isPlaygroundFullScreen;
	}

	$effect(() => {
		functionCallStackExampleImage =
			$mode === 'dark' ? StackFunctionCallStackExampleDarkSvg : StackFunctionCallStackExampleSvg;
	});
</script>

{#snippet Playground()}
	<div class="flex justify-between">
		<h3 class="m-0">Playground</h3>
		<div class="flex gap-2">
			<Button size="sm" onclick={() => stackPlaygroundRef?.reset()} class="gap-2"
				><ArrowClockwise />Reset</Button
			>
			<Button class="lg:hidden text-lg" size="sm" onclick={togglePlaygroundFullScreen}>
				{#if isPlaygroundFullScreen}
					<ArrowsInSimple />
				{:else}
					<ArrowsOutSimple />
				{/if}
			</Button>
		</div>
	</div>
	<hr class="m-0 mb-2" />

	<StackPlayground bind:this={stackPlaygroundRef} />
{/snippet}

{#if isPlaygroundFullScreen}
	<div class="fixed inset-0 bg-background z-10 flex justify-center w-full overflow-auto">
		<div class="py-8 px-4 flex flex-col gap-4 w-full prose">
			{@render Playground()}
		</div>
	</div>
{/if}

<div class={cn('prose min-w-full', isPlaygroundFullScreen && 'hidden')}>
	<BackArrowTitle href={CONST.ROUTES.DS()._()} title="Stack" />

	<div class="my-8"></div>

	<PageUnderDevelopment />

	<div class="space-y-6">
		<p>
			A stack is a linear data structure that follows the <b>Last In, First Out (LIFO)</b> principle.
			This means the last element added to the stack is the first one to be removed.
		</p>

		<div class="">
			<h3>Primary Operations:</h3>
			<ul>
				<li><b>Push:</b> Add an element to the top of the stack.</li>
				<li><b>Pop:</b> Remove the top element from the stack.</li>
			</ul>
		</div>

		<div class="">
			<h3>Secondary Methods:</h3>
			<ul>
				<li><b>Peek:</b> Get the top element of the stack <b>without removing it.</b></li>
				<li><b>isEmpty:</b> Check if the stack is empty.</li>
				<li><b>isFull:</b> Check if the stack is full.</li>
			</ul>
		</div>
	</div>

	<hr class="my-6" />

	<div class="space-y-6">
		<h2>Why Stack?</h2>
		<p>
			Stack is one of the most used data structures due to its <u>nature</u> of
			<b>Reverse ordering.</b> Let me explain:
		</p>
		<h3>Use Cases:</h3>
		<ul>
			<li>
				<b>Function Call Stack:</b> Every time a function is called, it is pushed onto the stack.
				When the function returns, it is popped off the stack.
				<div class="w-fit mx-auto">
					<img
						src={functionCallStackExampleImage}
						class="mb-0"
						alt="Stack: Function Call Stack Example"
					/>
					<p class="text-center text-xs">Function Call Stack Example</p>
				</div>
			</li>
			<li>
				<b>Undo Functionality:</b> In text editors, when you hit "Undo", the most recent change is reversed
				first. A stack keeps track of your actions so you can undo them in reverse order.
			</li>
		</ul>
	</div>

	<hr class="my-6" />

	<div>
		<h2>Core Idea</h2>
		<div>
			The main concept of a stack is the <b>TOS (Top of the Stack)</b>. It's a variable that keeps
			track of the top item in the stack.
			<ul>
				<li>TOS starts at <b>-1</b> because when the stack is empty, there's no top item.</li>
				<li>As you add items to the stack, TOS changes to show the index of the top item.</li>
			</ul>
			If the stack has items, TOS will tell you where the top item is. If it's -1, it means the stack
			is empty.

			<blockquote class="not-italic font-semibold">
				TOS is sometimes also referred as "head"
			</blockquote>
		</div>

		<div>By using TOS, we can easily implement stack using an Array.</div>
	</div>

	<div class="my-6"></div>

	<div class="px-4 py-4 border-2 border-dashed flex flex-col gap-4">
		{@render Playground()}
	</div>

	<h2>Stack Operations</h2>
	<div class="space-y-10">
		<div>
			<h3>Push</h3>
			<p>Adds an element on top of the stack.</p>
			<CodeBlock
				code={`
					void push(int data){
						if(isFull()){
							printf("Stack is full");
							return;
						}
						
						stack[++tos] = data;
					}
				`}
			/>

			<div>
				<h3>Algorithm:</h3>
				<ol>
					<li>Check if the stack is full. If it is, we print an error message and return.</li>
					<li>Increase TOS by 1 and assign the data at "newly increased" TOS index.</li>
				</ol>
			</div>
			<div>
				<h4>Why <code>stack[++tos] = data</code>?</h4>
				<p>
					As we want to add an element, we need to move the TOS forward by 1 index.
					<br />
					The operation is equivalent to: <code>stack[tos + 1] = data</code>
				</p>
			</div>
		</div>

		<div>
			<h3>Pop</h3>
			<p>Removes the top element from the stack.</p>
			<CodeBlock
				code={`
					int pop(){
						if(isEmpty()){
							printf("Stack is empty");
							return -1;
						}

						return stack[tos--];
					}
				`}
			/>

			<div>
				<h3>Algorithm:</h3>
				<ol>
					<li>Check if the stack is empty. If it is, we print an error message and return.</li>
					<li>Decrease the TOS by 1 and return the data from the old TOS index.</li>
				</ol>
			</div>
		</div>

		<div>
			<h3>isFull</h3>
			<p>Checks if the stack is full.</p>
			<CodeBlock
				code={`
					int isFull(){
						if(tos == MAX - 1){
							return true;
						}
						return false;
					}
				`}
			/>

			<p>
				For limited size stack, if the TOS reaches the maximum index, it means the stack is full.
			</p>

			<h4>Why <code>MAX - 1</code>?</h4>
			<p>
				MAX is the number of elements a stack can hold and here TOS is index which starts with 0, so
				to account that we need to subtract 1 from MAX.
			</p>
		</div>

		<div>
			<h3>isEmpty</h3>
			<p>Checks if the stack is empty.</p>
			<CodeBlock
				code={`
					int isEmpty(){
						if(tos == -1){
							return true;
						}
						return false;
					}
				`}
			/>

			<p>Fundamentally, we know that if TOS is -1, it means the stack is empty.</p>
		</div>

		<div>
			<h3>Peek</h3>
			<p>Returns the top element of the stack.</p>
			<CodeBlock
				code={`
					int peek(){
						if(isEmpty()){
							return -1;
						}
						return stack[tos];
					}
				`}
			/>

			<p>
				Condition is that if the stack is empty, TOS will be -1. So if we try to access <code
					>stack[-1]</code
				>, we will get error. In order to prevent that we are first checking if the stack is empty.
			</p>
		</div>
	</div>
</div>
