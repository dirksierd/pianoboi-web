<!-- Component for displaying and managing saved chords -->
<script lang="ts">
	import type { ChordSet, SavedChord, MatchingChords } from '$lib/utils/chords';
	import { findMatchingChords, generateScaleChords } from '$lib/utils/chords';
	import { createEventDispatcher } from 'svelte';
	import ChordDisplay from './ChordDisplay.svelte';
	import ChordScaleReference from './ChordScaleReference.svelte';
	import Piano from './Piano.svelte';
	import SheetMusic from './SheetMusic.svelte';

	export let chords: SavedChord[] = [];
	export let chordSets: ChordSet[] = [];
	export let currentSetId: string | null = null;
	export let currentChordId: string | null = null;
	export let playingChordId: string | null = null;
	export let viewMode: 'keyboard' | 'sheet' = 'keyboard';
	export let matchingChords: MatchingChords = { majorMatches: [], minorMatches: [] };

	const dispatch = createEventDispatcher<{
		setCurrentChord: { id: string | null };
		deleteChord: { id: string };
		playChord: { id: string; notes: any[] };
		assignToSet: { chordId: string; setId: string | null };
		clearChords: { setId: string | null };
	}>();

	// Get filtered chords based on current set selection
	$: filteredChords = currentSetId
		? chords.filter((chord) => chord.setId === currentSetId)
		: chords;

	// Get set name for display
	$: currentSetName = currentSetId
		? chordSets.find((set) => set.id === currentSetId)?.name || 'Unnamed Set'
		: 'All Chords';

	function handleSetCurrentChord(id: string | null) {
		dispatch('setCurrentChord', { id });
	}

	function handleDeleteChord(id: string) {
		dispatch('deleteChord', { id });
	}

	function handlePlayChord(chord: SavedChord) {
		dispatch('playChord', { id: chord.id, notes: chord.notes });
	}

	function handleAssignToSet(chordId: string, setId: string | null) {
		dispatch('assignToSet', { chordId, setId });
	}

	function handleClearChords() {
		dispatch('clearChords', { setId: currentSetId });
	}
</script>

{#if filteredChords && filteredChords.length > 0}
	<div class="space-y-2 p-2">
		<div class="header-content flex items-center justify-between">
			<h2 class="text-base font-bold text-gray-800">
				{currentSetName} ({filteredChords.length}
				{filteredChords.length === 1 ? 'chord' : 'chords'})
			</h2>

			<div class="flex gap-2">
				<button
					class="rounded bg-red-500 px-2 py-0.5 text-xs font-medium text-white hover:bg-red-600"
					on:click={handleClearChords}
				>
					Clear {currentSetId ? 'Set' : 'All'}
				</button>
			</div>
		</div>

		<!-- Insertion indicator at the beginning -->
		<div
			id="insert-marker-top"
			class="insertion-marker relative my-1 flex items-center justify-center"
			class:active={currentChordId === 'top'}
			class:hidden={currentChordId !== 'top'}
		>
			<div class="h-[2px] w-full rounded bg-blue-200"></div>
			<div
				class="absolute left-1/2 top-1/2 z-10 flex h-6 w-6 -translate-x-1/2 -translate-y-1/2 cursor-pointer items-center justify-center rounded-full bg-white shadow-sm hover:bg-blue-50"
				on:click={() => handleSetCurrentChord('top')}
			>
				<i class="fas fa-plus text-blue-500"></i>
			</div>
		</div>

		<!-- Saved chords - showing the primary view based on view mode -->
		{#each filteredChords as chord, index}
			<div
				id={chord.id}
				class="chord-container relative rounded-lg border bg-white p-2 shadow-sm transition-all duration-300 hover:shadow"
				class:bg-blue-50={playingChordId === chord.id}
			>
				<div class="mb-1 flex items-center justify-between text-xs text-gray-500">
					<div class="flex items-center gap-1">
						<div>
							Key: <span class="font-medium text-gray-700">{chord.signature.id}</span>
						</div>

						<!-- Set Assignment Control -->
						{#if chord.setId && chord.setId === currentSetId}
							<div class="flex items-center">
								<span class="ml-1 rounded-full bg-blue-100 px-1.5 py-0.5 text-xs text-blue-700">
									In Set: {chordSets.find((s) => s.id === chord.setId)?.name || 'Unknown'}
								</span>
								<button
									class="ml-1 text-xs text-blue-400 hover:text-blue-600"
									on:click={() => handleAssignToSet(chord.id, null)}
									title="Remove from set"
								>
									<svg
										xmlns="http://www.w3.org/2000/svg"
										class="h-3 w-3"
										viewBox="0 0 20 20"
										fill="currentColor"
									>
										<path
											fill-rule="evenodd"
											d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z"
											clip-rule="evenodd"
										/>
									</svg>
								</button>
							</div>
						{:else if chord.setId && chord.setId !== currentSetId}
							<div class="flex items-center">
								<span class="ml-1 rounded-full bg-gray-100 px-1.5 py-0.5 text-xs text-gray-600">
									In Set: {chordSets.find((s) => s.id === chord.setId)?.name || 'Unknown'}
								</span>
								{#if currentSetId}
									<button
										class="ml-1 text-xs text-blue-400 hover:text-blue-600"
										on:click={() => handleAssignToSet(chord.id, currentSetId)}
										title="Move to current set"
									>
										<svg
											xmlns="http://www.w3.org/2000/svg"
											class="h-3 w-3"
											viewBox="0 0 20 20"
											fill="currentColor"
										>
											<path
												d="M8 5a1 1 0 100 2h5.586l-1.293 1.293a1 1 0 001.414 1.414l3-3a1 1 0 000-1.414l-3-3a1 1 0 10-1.414 1.414L13.586 5H8z"
											/>
											<path
												d="M12 15a1 1 0 100-2H6.414l1.293-1.293a1 1 0 10-1.414-1.414l-3 3a1 1 0 000 1.414l3 3a1 1 0 001.414-1.414L6.414 15H12z"
											/>
										</svg>
									</button>
								{/if}
							</div>
						{:else if currentSetId}
							<button
								class="ml-1 rounded-full bg-gray-100 px-1.5 py-0.5 text-xs text-gray-600 hover:bg-blue-100 hover:text-blue-700"
								on:click={() => handleAssignToSet(chord.id, currentSetId)}
							>
								<span class="flex items-center gap-1">
									<svg
										xmlns="http://www.w3.org/2000/svg"
										class="h-3 w-3"
										viewBox="0 0 20 20"
										fill="currentColor"
									>
										<path
											d="M5 3a2 2 0 00-2 2v2a2 2 0 002 2h2a2 2 0 002-2V5a2 2 0 00-2-2H5zM5 11a2 2 0 00-2 2v2a2 2 0 002 2h2a2 2 0 002-2v-2a2 2 0 00-2-2H5zM11 5a2 2 0 012-2h2a2 2 0 012 2v2a2 2 0 01-2 2h-2a2 2 0 01-2-2V5zM14 11a1 1 0 011 1v1h1a1 1 0 110 2h-1v1a1 1 0 11-2 0v-1h-1a1 1 0 110-2h1v-1a1 1 0 011-1z"
										/>
									</svg>
									Add to set
								</span>
							</button>
						{/if}
					</div>
				</div>

				<!-- Delete button - positioned at top right -->
				<button
					class="absolute right-2 top-1 text-gray-400 hover:text-red-500"
					on:click={() => handleDeleteChord(chord.id)}
					aria-label="Delete chord"
				>
					<i class="fas fa-times"></i>
				</button>

				<!-- Primary visualization based on view mode -->
				<div class="flex w-full">
					<div class="flex-1 overflow-hidden">
						{#if viewMode === 'keyboard'}
							<div class="overflow-x-auto">
								<Piano
									notes={chord.notes}
									readonly={true}
									compact={true}
									showLabels={true}
									signature={chord.signature}
								/>
							</div>
						{:else}
							<div class="flex flex-col gap-2 sm:flex-row">
								<!-- Left: Sheet Music -->
								<div class="flex-1">
									<SheetMusic notes={chord.notes} signature={chord.signature} />
								</div>

								<!-- Right: Chord Display Table -->
								<div class="flex flex-1 flex-col gap-2">
									<ChordDisplay notes={chord.notes} signature={chord.signature} debug={false} />

									<!-- Scale Degree Reference Table -->
									<ChordScaleReference
										signature={chord.signature}
										notes={chord.notes}
										detectedChords={[]}
										matchingChords={findMatchingChords(
											chord.notes,
											generateScaleChords(chord.signature)
										)}
										hasInfoButton={false}
									/>
								</div>
							</div>
						{/if}
					</div>
				</div>

				<!-- Play button - positioned at bottom right -->
				<button
					class="absolute bottom-2 right-2 flex items-center rounded-full bg-blue-500 mx-1 my-1 px-2 py-1 text-white transition-colors hover:bg-blue-600"
					class:animate-pulse={playingChordId === chord.id}
					on:click={() => handlePlayChord(chord)}
					aria-label="Play chord"
					title="Play chord (or press 'P' to play latest chord)"
				>
					<i class="fas fa-play mr-1 text-xs"></i>
					<span class="text-xs">Play</span>
				</button>
			</div>

			<!-- Insertion marker after each chord -->
			<div
				id={`insert-marker-${chord.id}`}
				class="insertion-marker relative my-1 flex items-center justify-center"
				class:active={currentChordId === chord.id}
				class:hidden={currentChordId !== chord.id}
			>
				<div class="h-[2px] w-full rounded bg-blue-200"></div>
				<div
					class="absolute left-1/2 top-1/2 z-10 flex h-6 w-6 -translate-x-1/2 -translate-y-1/2 cursor-pointer items-center justify-center rounded-full bg-white shadow-sm hover:bg-blue-50"
					on:click={() => handleSetCurrentChord(chord.id)}
				>
					<i class="fas fa-plus text-blue-500"></i>
				</div>
			</div>
		{/each}
	</div>
{:else}
	<div class="flex h-full items-center justify-center">
		<div class="text-center text-gray-500">
			<p class="mb-1 text-base font-medium">No saved chords</p>
			<p class="text-xs">Press the space bar to save your current chord</p>
		</div>
	</div>
{/if}

<style>
	.chord-container {
		margin-bottom: 0.5rem;
	}

	.insertion-marker {
		height: 1.5rem;
		position: relative;
		margin: 0.25rem 0;
	}

	.insertion-marker.active .insertion-line {
		@apply h-[2px] bg-blue-300;
	}
</style>
