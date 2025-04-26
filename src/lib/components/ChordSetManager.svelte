<!-- Component for managing chord sets -->
<script lang="ts">
	import type { ChordSet } from '$lib/utils/chords';
	import { createEventDispatcher } from 'svelte';

	export let chordSets: ChordSet[] = [];
	export let currentSetId: string | null = null;
	export let savedChordCount: number = 0;
	export let showCreate: boolean = true;

	const dispatch = createEventDispatcher<{
		select: { setId: string | null };
		delete: { setId: string };
		create: { name: string };
	}>();

	let isSetMenuOpen = false;
	let newSetName = '';

	function toggleMenu() {
		isSetMenuOpen = !isSetMenuOpen;
	}

	function handleSelectSet(setId: string | null) {
		dispatch('select', { setId });
		isSetMenuOpen = false;
	}

	function handleDeleteSet(setId: string, event: MouseEvent) {
		event.stopPropagation();
		dispatch('delete', { setId });
	}

	function handleCreateSet() {
		if (!newSetName.trim()) return;

		dispatch('create', { name: newSetName.trim() });
		newSetName = '';
		isSetMenuOpen = false;
	}

	// Get set name for display
	$: currentSetName = currentSetId
		? chordSets.find((set) => set.id === currentSetId)?.name || 'Unnamed Set'
		: 'All Chords';
</script>

<div class="relative">
	<button
		class="flex items-center gap-1 rounded-lg border bg-white px-2 py-1 text-xs font-medium shadow-sm hover:bg-gray-50"
		on:click={toggleMenu}
	>
		<!-- Folder Icon -->
		<svg
			xmlns="http://www.w3.org/2000/svg"
			class="h-3.5 w-3.5 text-blue-600"
			viewBox="0 0 20 20"
			fill="currentColor"
		>
			<path d="M2 6a2 2 0 012-2h5l2 2h5a2 2 0 012 2v6a2 2 0 01-2 2H4a2 2 0 01-2-2V6z" />
		</svg>
		<span class="font-medium text-gray-700">Set:</span>
		<span class="ml-1 text-blue-600">{currentSetName}</span>
		<svg
			class="ml-1 h-3.5 w-3.5"
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

	{#if isSetMenuOpen}
		<div
			class="absolute left-0 top-full z-30 mt-1 w-72 rounded-md border border-gray-200 bg-white p-2 shadow-lg"
		>
			<div class="mb-1 border-b pb-1 text-xs font-medium text-gray-700">Chord Sets</div>

			<!-- Set List -->
			<div class="mb-2 max-h-52 overflow-y-auto">
				<button
					class="mb-1 flex w-full items-center justify-between rounded-md px-2 py-1 text-left text-xs hover:bg-blue-50"
					class:bg-blue-100={currentSetId === null}
					on:click={() => handleSelectSet(null)}
				>
					<span class="font-medium">All Chords</span>
					<span class="text-xs text-gray-500">{savedChordCount} chords</span>
				</button>

				{#each chordSets as set}
					<div
						class="flex w-full items-center justify-between rounded-md px-1 py-0.5 hover:bg-blue-50"
						class:bg-blue-100={currentSetId === set.id}
					>
						<button
							class="flex-1 px-2 py-0.5 text-left text-xs"
							on:click={() => handleSelectSet(set.id)}
						>
							<span class="font-medium">{set.name}</span>
							<span class="ml-1 text-[10px] text-gray-500">
								{chordSets.filter((c) => c.id === set.id).length} chords
							</span>
						</button>
						<button
							class="ml-1 rounded p-0.5 text-red-400 hover:bg-red-50 hover:text-red-600"
							on:click={(e) => handleDeleteSet(set.id, e)}
						>
							<svg
								xmlns="http://www.w3.org/2000/svg"
								class="h-3.5 w-3.5"
								viewBox="0 0 20 20"
								fill="currentColor"
							>
								<path
									fill-rule="evenodd"
									d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z"
									clip-rule="evenodd"
								/>
							</svg>
						</button>
					</div>
				{/each}
			</div>

			<!-- Create New Set Form -->
			{#if showCreate}
				<div class="border-t pt-1">
					<form on:submit|preventDefault={handleCreateSet} class="flex items-center">
						<input
							type="text"
							bind:value={newSetName}
							placeholder="New set name..."
							class="flex-1 rounded-md border-gray-300 px-2 py-1 text-xs shadow-sm focus:border-blue-300 focus:ring focus:ring-blue-200 focus:ring-opacity-50"
						/>
						<button
							type="submit"
							disabled={!newSetName.trim()}
							class="ml-1 rounded-md bg-blue-500 px-2 py-1 text-xs font-medium text-white shadow-sm hover:bg-blue-600 disabled:opacity-50"
						>
							Create
						</button>
					</form>
				</div>
			{/if}
		</div>
	{/if}
</div>
