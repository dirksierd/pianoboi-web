<!-- Component for selecting MIDI devices -->
<script lang="ts">
	import { createEventDispatcher } from 'svelte';
	import type { Input } from 'webmidi';

	export let midiEnabled: boolean = false;
	export let midiInputs: Input[] = [];
	export let selectedInput: Input | null = null;
	export let midiError: string = '';
	export let isInitializing: boolean = false;

	const dispatch = createEventDispatcher<{
		select: { input: Input };
		refresh: void;
	}>();

	let isMenuOpen = false;

	function toggleMenu() {
		isMenuOpen = !isMenuOpen;
	}

	function handleInputChange(input: Input) {
		dispatch('select', { input });
		isMenuOpen = false;
	}

	function refreshMidiDevices() {
		dispatch('refresh');
	}
</script>

<div class="relative">
	<button
		class="flex items-center gap-1.5 rounded-lg border bg-white px-3 py-1.5 text-sm font-medium shadow hover:bg-gray-50"
		on:click={toggleMenu}
	>
		<svg
			xmlns="http://www.w3.org/2000/svg"
			class="h-4 w-4 text-blue-600"
			viewBox="0 0 20 20"
			fill="currentColor"
		>
			<path
				fill-rule="evenodd"
				d="M7 2a1 1 0 00-.707 1.707L7 4.414v3.758a1 1 0 01-.293.707l-4 4C.817 14.769 2.156 18 4.828 18h10.343c2.673 0 4.012-3.231 2.122-5.121l-4-4A1 1 0 0113 8.172V4.414l.707-.707A1 1 0 0013 2H7zm2 6.172V4h2v4.172a3 3 0 00.879 2.12l1.168 1.168a4 4 0 00-2.324.971l-.243.212a4 4 0 01-5.113.44l-.177-.156a4 4 0 00-2.303-.988l1.168-1.168A3 3 0 009 8.172z"
				clip-rule="evenodd"
			/>
		</svg>
		{selectedInput ? selectedInput.name : 'Select MIDI Device'}
		<svg
			class="ml-1 h-4 w-4"
			fill="currentColor"
			viewBox="0 0 20 20"
			xmlns="http://www.w3.org/2000/svg"
		>
			<path
				fill-rule="evenodd"
				d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z"
				clip-rule="evenodd"
			></path>
		</svg>
	</button>

	<!-- Dropdown menu -->
	{#if isMenuOpen}
		<div
			class="dropdown-menu absolute right-0 mt-2 w-56 rounded-md border border-gray-200 bg-white shadow-lg"
			style="z-index: 9999;"
		>
			<div class="p-2">
				<div class="mb-2 flex justify-between border-b pb-2">
					<span class="font-medium text-gray-700">MIDI Devices</span>
					<button
						class="text-xs text-blue-600 hover:text-blue-800"
						on:click={refreshMidiDevices}
						disabled={isInitializing}
					>
						{isInitializing ? 'Refreshing...' : 'Refresh Devices'}
					</button>
				</div>

				{#if midiInputs.length === 0}
					<div class="py-2 text-center text-sm text-gray-500">No MIDI devices found</div>
				{:else}
					<div class="max-h-40 overflow-y-auto py-1">
						{#each midiInputs as input}
							<button
								class="flex w-full items-center rounded px-3 py-2 text-left text-sm hover:bg-gray-100"
								class:bg-blue-50={selectedInput === input}
								class:font-medium={selectedInput === input}
								class:text-blue-700={selectedInput === input}
								on:click={() => handleInputChange(input)}
							>
								{input.name}
							</button>
						{/each}
					</div>
				{/if}
			</div>
		</div>
	{/if}

	<!-- Error Message -->
	{#if !midiEnabled && midiError}
		<div class="mt-2 rounded border border-red-200 bg-red-50 px-3 py-2 text-sm text-red-700">
			<p>{midiError}</p>
		</div>
	{/if}
</div>
