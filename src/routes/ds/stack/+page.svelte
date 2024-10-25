<script lang="ts">
	import BackArrowTitle from '$lib/components/BackArrowTitle.svelte';
	import CodeBlock from '$lib/components/CodeBlock.svelte';
	import StackPlayground from '$lib/components/playgrounds/Stack.svelte';
	import StackFunctionCallStackExampleSvg from '$lib/assets/svg/stack-function-call-stack-example.svg';
	import { CONST } from '$lib/const';
	import { Warning } from 'phosphor-svelte';
	import { Button } from '$lib/components/ui/button';
	import { ArrowClockwise } from 'phosphor-svelte';
	import PageUnderDevelopment from '$lib/components/PageUnderDevelopment.svelte';

	let stackPlaygroundRef: ReturnType<typeof StackPlayground> | undefined = $state();
</script>

<div class="prose min-w-full">
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
						src={StackFunctionCallStackExampleSvg}
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

			<blockquote class="not-italic font-semibold">TOS is sometimes also called head</blockquote>
		</div>

		<div>By using TOS, we can easily implement stack using an Array.</div>
	</div>

	<div class="my-6"></div>

	<div class="px-4 py-4 border-2 border-dashed flex flex-col gap-4">
		<div class="flex justify-between">
			<h3 class="m-0">Playground</h3>
			<Button size="sm" onclick={() => stackPlaygroundRef?.reset()} class="gap-2"
				><ArrowClockwise />Reset</Button
			>
		</div>
		<hr class="m-0 mb-2" />

		<StackPlayground bind:this={stackPlaygroundRef} />
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
	</div>
</div>
