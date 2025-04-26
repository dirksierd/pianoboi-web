<!-- PianoBoi - An interactive piano with chord detection and saving -->
<script lang="ts">
	import { browser } from '$app/environment';
	import type { Signature } from '$lib/types/signatures';
	import { signatures } from '$lib/types/signatures';
	import { Chord } from '@tonaljs/tonal';
	import { onDestroy, onMount } from 'svelte';
	import type { Input, NoteMessageEvent } from 'webmidi';
	// Import components
	import ChordDisplay from './ChordDisplay.svelte';
	import ChordScaleReference from './ChordScaleReference.svelte';
	import ChordSetManager from './ChordSetManager.svelte';
	import KeySignatureSelector from './KeySignatureSelector.svelte';
	import MidiDeviceSelector from './MidiDeviceSelector.svelte';
	import Piano from './Piano.svelte';
	import SavedChordsList from './SavedChordsList.svelte';
	import SheetMusic from './SheetMusic.svelte';
	import ViewModeSwitcher from './ViewModeSwitcher.svelte';
	// Import utilities
	import {
		calculateMidiFromName,
		calculateNoteNumber,
		initAudio,
		loadPianoSamples,
		playNote,
		stopNote
	} from '$lib/utils/audio';
	import {
		assignChordToSet,
		type ChordSet,
		createChordSet,
		deleteChordSet,
		extractPitchClasses,
		findMatchingChords,
		generateScaleChords,
		loadSavedChords,
		type MatchingChords,
		persistSavedChords,
		type SavedChord
	} from '$lib/utils/chords';
	import { disableWebMidi, initializeWebMidi, setupMidiListeners } from '$lib/utils/midi';

	// State
	let midiEnabled = false;
	let midiInputs: Input[] = [];
	let selectedInput: Input | null = null;
	let activeNotes: any[] = [];
	let currentSignature: Signature = signatures[0];
	let midiError = '';
	let isInitializing = false;
	let isMenuOpen = false;
	let currentChords: string[] = [];

	// Audio synthesis
	let audioContext: AudioContext | null = null;
	let activeAudioNodes: Record<string, any> = {};
	let playingAudioNodes: Array<{ note: number; nodes: any }> = [];
	let samplesLoaded = false;
	let loadingProgress = 0;

	// Piano samples cache
	let pianoSamples: { [key: string]: AudioBuffer } = {};

	// View mode state
	let viewMode: 'keyboard' | 'sheet' = 'keyboard';

	// Chord management state
	let savedChords: SavedChord[] = [];
	let chordSets: ChordSet[] = [];
	let currentSetId: string | null = null;
	let chordsContainerElement: HTMLDivElement;
	let currentChordId: string | null = null;
	let playingChordId: string | null = null;

	// State for chord matching
	let matchingChords: MatchingChords = { majorMatches: [], minorMatches: [] };

	// Load saved chords and sets from localStorage
	onMount(() => {
		const result = loadSavedChords();
		savedChords = result.savedChords;
		chordSets = result.chordSets;

		initMidi();
		initAudioContext();

		if (browser) {
			window.addEventListener('keydown', handleKeyDown);
		}
	});

	onDestroy(() => {
		// Disable WebMidi (this cleans up all listeners)
		disableWebMidi();

		// Clean up audio resources
		clearActiveNotes();
		if (audioContext) {
			audioContext.close().catch((e) => console.error('Error closing audio context:', e));
		}

		// Remove keyboard listener - only in browser environment
		if (browser) {
			window.removeEventListener('keydown', handleKeyDown);
		}
	});

	// Initialize audio context
	async function initAudioContext() {
		audioContext = initAudio();
		if (audioContext) {
			pianoSamples = await loadPianoSamples(audioContext, (progress) => {
				loadingProgress = progress;
				samplesLoaded = progress === 100;
			});
			samplesLoaded = Object.keys(pianoSamples).length > 0;
		}
	}

	// Initialize MIDI
	async function initMidi() {
		isInitializing = true;
		const result = await initializeWebMidi();
		midiEnabled = result.midiEnabled;
		midiInputs = result.midiInputs;
		midiError = result.midiError;

		// Auto-select first input if available
		if (midiInputs.length > 0 && !selectedInput) {
			selectedInput = midiInputs[0];
			setupMidiInput();
		}

		isInitializing = false;
	}

	// MIDI event handlers
	function handleNoteOn(e: NoteMessageEvent) {
		console.log('Note ON:', e.note);

		// Create a standardized note object that's compatible with both MIDI and onscreen keyboard
		const note = {
			name: e.note.name,
			accidental: e.note.accidental || '',
			octave: e.note.octave,
			number: e.note.number,
			identifier: `${e.note.name}${e.note.accidental || ''}${e.note.octave}`,
			attack: e.note.attack || 0.5,
			release: e.note.release || 0.5
		};

		console.log('Standardized MIDI note:', note);
		activeNotes = [...activeNotes, note];

		// Update matching chords
		matchingChords = findMatchingChords(activeNotes, generateScaleChords(currentSignature));
	}

	function handleNoteOff(e: NoteMessageEvent) {
		console.log('Note OFF:', e.note);
		// Filter using the note number and octave which are the most reliable identifiers
		activeNotes = activeNotes.filter(
			(note) => !(note.number === e.note.number && note.octave === e.note.octave)
		);
	}

	function handleInputChanged(event: CustomEvent<{ input: Input }>) {
		selectedInput = event.detail.input;
		setupMidiInput();
	}

	function setupMidiInput() {
		setupMidiListeners(selectedInput, handleNoteOn, handleNoteOff);
	}

	function refreshMidiDevices() {
		if (selectedInput) {
			selectedInput = null;
			activeNotes = [];
		}

		initMidi();
	}

	// Handle piano key presses coming from the Piano component
	function handlePianoNotePress(event: {
		detail: { name: string; accidental: string; octave: number; isOn: boolean };
	}) {
		const noteData = event.detail;
		console.log('Piano key event:', noteData);

		if (noteData.isOn) {
			// Simulate note-on
			// Create a WebMidi compatible note object
			const note = {
				name: noteData.name.toUpperCase(), // WebMidi uses uppercase notes
				accidental: noteData.accidental,
				octave: noteData.octave,
				// Calculate MIDI note number (middle C = 60)
				number: calculateNoteNumber(noteData.name, noteData.accidental, noteData.octave),
				// Additional properties that might be needed
				identifier: `${noteData.name}${noteData.accidental}${noteData.octave}`,
				attack: 0.5,
				release: 0.5
			};

			console.log('Created WebMidi note object:', note);
			activeNotes = [...activeNotes, note];

			// Play the note
			if (audioContext) {
				const nodes = playNote(note.number, audioContext, pianoSamples, samplesLoaded);
				if (nodes) {
					activeAudioNodes[note.identifier] = nodes;
				}
			}

			// Update matching chords
			matchingChords = findMatchingChords(activeNotes, generateScaleChords(currentSignature));
		} else {
			// Simulate note-off
			const noteToRemove = activeNotes.find(
				(note) =>
					note.name === noteData.name.toUpperCase() &&
					note.accidental === noteData.accidental &&
					note.octave === noteData.octave
			);

			// Stop the audio for this note
			if (noteToRemove) {
				stopNote(activeAudioNodes[noteToRemove.identifier]);
				delete activeAudioNodes[noteToRemove.identifier];
			}

			activeNotes = activeNotes.filter(
				(note) =>
					!(
						note.name === noteData.name.toUpperCase() &&
						note.accidental === noteData.accidental &&
						note.octave === noteData.octave
					)
			);

			// Update matching chords
			matchingChords = findMatchingChords(activeNotes, generateScaleChords(currentSignature));
		}
	}

	// Chord saving functionality
	function saveCurrentChord() {
		console.log('Attempting to save chord:', activeNotes);
		// Don't save if no notes are active
		if (activeNotes.length === 0) {
			console.log('No notes to save');
			return;
		}

		const id = `chord-${Date.now()}`;

		// Process notes to ensure consistent format before saving
		const processedNotes = activeNotes.map((note) => {
			// Check if this is a WebMidi note (which has _name, _accidental properties)
			// or a manually created note (which has name, accidental properties)
			const processedNote = {
				name: note.name || note._name,
				accidental: note.accidental || note._accidental || '',
				octave: note.octave || note._octave,
				number:
					note.number ||
					calculateNoteNumber(
						note.name || note._name,
						note.accidental || note._accidental || '',
						note.octave || note._octave
					),
				identifier:
					note.identifier ||
					`${note.name || note._name}${note.accidental || note._accidental || ''}${
						note.octave || note._octave
					}`,
				attack: note.attack || note._attack || 0.5,
				release: note.release || note._release || 0.5,
				signature: currentSignature
			};

			console.log('Processed note for saving:', processedNote);
			return processedNote;
		});

		const newChord: SavedChord = {
			id,
			notes: processedNotes,
			signature: currentSignature,
			timestamp: Date.now(),
			setId: currentSetId
		};

		console.log('Creating new chord:', newChord);

		// Insert the chord at the appropriate position
		if (currentChordId === 'top') {
			savedChords = [newChord, ...savedChords];
		} else if (currentChordId) {
			const index = savedChords.findIndex((chord) => chord.id === currentChordId);
			if (index !== -1) {
				const updatedChords = [...savedChords];
				updatedChords.splice(index + 1, 0, newChord);
				savedChords = updatedChords;
			} else {
				savedChords = [...savedChords, newChord];
			}
		} else {
			// No insertion point set, add to end
			savedChords = [...savedChords, newChord];
		}

		// Debug log to check that the chord was added
		console.log('Saved chords now:', savedChords);

		// Set the current chord ID to the new chord's ID
		// so next insertion will be after this chord
		setCurrentChord(id);

		// Persist saved chords
		persistSavedChords(savedChords, chordSets);
	}

	// Function to set the current chord and update the insertion marker
	function setCurrentChord(id: string | null): void {
		currentChordId = id;

		// Use a small delay to ensure DOM is updated before scrolling
		setTimeout(() => {
			if (id) {
				// For insertion markers
				const markerId = id === 'top' ? 'insert-marker-top' : `insert-marker-${id}`;
				const marker = document.getElementById(markerId);
				if (marker) {
					console.log(`Scrolling to marker: ${markerId}`);
					marker.scrollIntoView({ behavior: 'smooth', block: 'center' });
				}

				// For the chord itself
				if (id !== 'top') {
					const chordElement = document.getElementById(id);
					if (chordElement) {
						console.log(`Scrolling to chord: ${id}`);
						chordElement.scrollIntoView({ behavior: 'smooth', block: 'center' });
						// Add some visual feedback using Tailwind classes
						chordElement.classList.add('ring-2', 'ring-blue-500', 'ring-opacity-50');
						setTimeout(() => {
							chordElement.classList.remove('ring-2', 'ring-blue-500', 'ring-opacity-50');
						}, 1500);
					}
				}
			}
		}, 50);
	}

	// Function for clearing all active notes (used when deleting a chord)
	function clearActiveNotes() {
		// Clear the active notes array
		activeNotes = [];

		// Stop any playing audio
		if (playingAudioNodes.length > 0) {
			playingAudioNodes.forEach((item) => {
				stopNote(item.nodes);
			});
			playingAudioNodes = [];
		}

		if (Object.keys(activeAudioNodes).length > 0) {
			Object.values(activeAudioNodes).forEach((nodes) => {
				stopNote(nodes);
			});
			activeAudioNodes = {};
		}
	}

	// Chord set management
	function handleCreateSet(event: CustomEvent<{ name: string }>) {
		const { newSet, updatedSets } = createChordSet(event.detail.name, chordSets);
		chordSets = updatedSets;
		currentSetId = newSet.id;
		persistSavedChords(savedChords, chordSets);
	}

	function handleSelectSet(event: CustomEvent<{ setId: string | null }>) {
		currentSetId = event.detail.setId;
	}

	function handleDeleteSet(event: CustomEvent<{ setId: string }>) {
		const { updatedSets, updatedChords, newCurrentSetId } = deleteChordSet(
			event.detail.setId,
			chordSets,
			savedChords,
			currentSetId
		);

		chordSets = updatedSets;
		savedChords = updatedChords;
		currentSetId = newCurrentSetId;

		persistSavedChords(savedChords, chordSets);
	}

	// Chord management
	function handleDeleteChord(event: CustomEvent<{ id: string }>) {
		const id = event.detail.id;

		// If the chord being deleted is currently playing, stop it
		if (playingChordId === id) {
			stopChord();
		}

		// Remove the chord from the array
		savedChords = savedChords.filter((chord) => chord.id !== id);
		persistSavedChords(savedChords, chordSets);

		// Clear any notes that are currently playing
		clearActiveNotes();

		// Update current chord ID if needed
		if (currentChordId === id) {
			const chordIndex = savedChords.findIndex((chord) => chord.id === id);
			if (chordIndex > 0) {
				currentChordId = savedChords[chordIndex - 1].id;
			} else if (savedChords.length > 0) {
				currentChordId = 'top';
			} else {
				currentChordId = null;
			}
		}
	}

	function handleClearChords(event: CustomEvent<{ setId: string | null }>) {
		const setId = event.detail.setId;

		if (setId) {
			// Only clear chords in current set
			savedChords = savedChords.filter((chord) => chord.setId !== setId);
		} else {
			// Clear all chords
			savedChords = [];
		}

		persistSavedChords(savedChords, chordSets);
		currentChordId = null;
	}

	function handleAssignToSet(event: CustomEvent<{ chordId: string; setId: string | null }>) {
		const { chordId, setId } = event.detail;
		savedChords = assignChordToSet(chordId, setId, savedChords);
		persistSavedChords(savedChords, chordSets);
	}

	// Function to play a chord
	function handlePlayChord(event: CustomEvent<{ id: string; notes: any[] }>) {
		const { id, notes } = event.detail;

		// Stop previous playing chord if any
		if (playingChordId) {
			stopChord();
		}

		console.log(`Playing chord ${id}:`, notes);
		playingChordId = id;

		// Stagger note timing slightly for more natural sound
		const audioNodes: any[] = [];
		notes.forEach((note, index) => {
			setTimeout(() => {
				// Extract note information and convert to MIDI
				let midiNote: number;

				if (typeof note === 'number') {
					// Already a MIDI number
					midiNote = note;
				} else if (note.number !== undefined) {
					// Has a direct MIDI number property
					midiNote = note.number;
				} else if (note.name !== undefined) {
					// Has note name, accidental, and octave - calculate MIDI
					const noteName = note.name + (note.accidental || '');
					const octave = note.octave;
					console.log(`Converting note ${noteName}${octave} to MIDI`);
					midiNote = calculateMidiFromName(noteName, octave);
				} else {
					console.error('Unrecognized note format:', note);
					return;
				}

				// Ensure MIDI note is valid (piano range is 21-108)
				if (typeof midiNote !== 'number' || isNaN(midiNote)) {
					console.error(`Invalid MIDI note number: ${midiNote}`);
					return;
				}

				// Clamp to valid piano range
				const clampedMidiNote = Math.max(21, Math.min(108, midiNote));
				if (clampedMidiNote !== midiNote) {
					console.log(`Clamped MIDI note ${midiNote} to ${clampedMidiNote} (valid piano range)`);
					midiNote = clampedMidiNote;
				}

				console.log(`Playing chord note with MIDI number: ${midiNote}`);
				if (audioContext) {
					const nodes = playNote(midiNote, audioContext, pianoSamples, samplesLoaded);
					if (nodes) {
						audioNodes.push({ note: midiNote, nodes });
						playingAudioNodes.push({ note: midiNote, nodes });
					}
				}
			}, index * 10); // 10ms stagger between notes
		});
	}

	// Function to stop the currently playing chord
	function stopChord() {
		if (playingAudioNodes.length > 0) {
			playingAudioNodes.forEach((item) => {
				stopNote(item.nodes);
			});
			playingAudioNodes = [];
		}
		playingChordId = null;
	}

	// Listen for keyboard shortcuts
	function handleKeyDown(event: KeyboardEvent) {
		// Space to save chord
		if (event.code === 'Space' && !event.repeat) {
			event.preventDefault(); // Prevent scrolling
			saveCurrentChord();
		}

		// 'P' key to play the most recent chord
		if (event.code === 'KeyP' && !event.repeat && savedChords.length > 0) {
			event.preventDefault();
			// Get the most recent chord
			const lastChord = savedChords[savedChords.length - 1];
			handlePlayChord({ detail: { id: lastChord.id, notes: lastChord.notes } } as CustomEvent<{
				id: string;
				notes: any[];
			}>);
		}
	}

	// Update matching chords whenever activeNotes changes
	$: scaleChords = generateScaleChords(currentSignature);
	$: matchingChords = findMatchingChords(activeNotes, scaleChords);

	// Update currentChords whenever activeNotes changes
	$: if (activeNotes) {
		const pitchClasses = extractPitchClasses(activeNotes);
		currentChords = Chord.detect(pitchClasses);
	}
</script>

<!-- Main Layout with Sticky UI -->
<div class="flex h-screen flex-col">
	<!-- Sticky top navigation -->
	<header class="sticky top-0 z-30 bg-white shadow-md">
		<div class="container mx-auto p-2">
			<div class="header-content flex items-center justify-between">
				<!-- Title and left controls -->
				<div class="header-content flex items-center gap-2">
					<!-- View mode toggle -->
					<ViewModeSwitcher {viewMode} on:change={(e) => (viewMode = e.detail.mode)} />

					<!-- Key Signature Dropdown -->
					<KeySignatureSelector
						{currentSignature}
						on:select={(e) => (currentSignature = e.detail.signature)}
					/>

					<!-- Chord Set Selector Dropdown -->
					<ChordSetManager
						{chordSets}
						{currentSetId}
						savedChordCount={savedChords.length}
						on:select={handleSelectSet}
						on:delete={handleDeleteSet}
						on:create={handleCreateSet}
					/>
				</div>

				<!-- Right-aligned MIDI Device Dropdown -->
				<MidiDeviceSelector
					{midiEnabled}
					{midiInputs}
					{selectedInput}
					{midiError}
					{isInitializing}
					on:select={handleInputChanged}
					on:refresh={refreshMidiDevices}
				/>
			</div>
		</div>
	</header>

	<!-- Fixed layout with scrollable content and fixed bottom keyboard -->
	<div class="flex flex-1 overflow-hidden">
		<!-- Main scrollable content area -->
		<div class="flex-1 overflow-hidden">
			<!-- Scrollable saved chords -->
			<div
				bind:this={chordsContainerElement}
				class="h-full overflow-y-auto pb-[240px]"
				id="chord-container"
			>
				<!-- Saved chords content -->
				<SavedChordsList
					chords={savedChords}
					{chordSets}
					{currentSetId}
					{currentChordId}
					{playingChordId}
					{viewMode}
					on:setCurrentChord={(e) => setCurrentChord(e.detail.id)}
					on:deleteChord={handleDeleteChord}
					on:playChord={handlePlayChord}
					on:assignToSet={handleAssignToSet}
					on:clearChords={handleClearChords}
				/>
			</div>
		</div>
	</div>

	<!-- Focus button that floats in the bottom-right corner -->
	<button
		class="focus-icon"
		on:click={() => {
			// If there's an active insertion point, scroll to it
			if (currentChordId) {
				const markerId =
					currentChordId === 'top' ? 'insert-marker-top' : `insert-marker-${currentChordId}`;
				const marker = document.getElementById(markerId);
				if (marker) {
					marker.scrollIntoView({ behavior: 'smooth', block: 'center' });
				}
			} else {
				// Otherwise, focus on the bottom of the list to add a new chord at the end
				if (chordsContainerElement) {
					chordsContainerElement.scrollTop = chordsContainerElement.scrollHeight;
				}
			}
		}}
		aria-label="Focus on current insertion point"
	>
		<i class="fas fa-crosshairs text-xl"></i>
	</button>

	<!-- Sticky player UI at the bottom of the screen - always shows both keyboard and sheet music -->
	<div
		class="sticky bottom-0 z-20 border-t bg-white pb-2 pt-1 shadow-[0_-4px_6px_-1px_rgba(0,0,0,0.1)]"
	>
		<div class="container mx-auto px-4">
			<div class="mb-2 flex items-center justify-between">
				<div>
					<!-- Save button -->
					<button
						class="flex items-center gap-1 rounded-lg bg-blue-500 px-2 py-1 text-xs font-medium text-white shadow-sm transition-all duration-150 hover:bg-blue-600 hover:shadow-md active:scale-95 active:bg-blue-700"
						on:click={saveCurrentChord}
						title="Save the current chord (Space)"
					>
						<i class="fas fa-save"></i>
						<span class="ml-1">Save Chord</span>
						<span class="ml-1 rounded bg-blue-600/80 px-1 py-0.5 text-xs font-medium">Space</span>
					</button>
				</div>

				<!-- Current insertion point indicator -->
				{#if currentChordId}
					<div class="flex items-center rounded-full bg-blue-50 px-2 py-0.5 text-xs text-blue-700">
						<span>
							{#if currentChordId === 'top'}
								Adding at beginning
							{:else}
								Adding after selected chord
							{/if}
						</span>
						<button
							class="ml-1 rounded-full bg-blue-100 p-1 text-blue-700 hover:bg-blue-200"
							on:click={() => setCurrentChord(null)}
						>
							<i class="fas fa-times text-xs"></i>
						</button>
					</div>
				{:else}
					<div class="text-xs text-gray-500">Adding at end</div>
				{/if}

				<div>
					<!-- Currently playing indicator -->
					<span class="flex items-center text-xs font-medium text-gray-600">
						{#if activeNotes.length > 0}
							<span class="mr-2 inline-block h-2 w-2 animate-pulse rounded-full bg-green-500"
							></span>
							{activeNotes.length} notes playing
						{:else}
							<span class="mr-2 inline-block h-2 w-2 rounded-full bg-gray-300"></span>
							No notes playing
						{/if}
					</span>
				</div>
			</div>

			<!-- Sheet music display with chord reference table -->
			<div class="mb-2 min-h-[70px] border-b pb-2">
				<!-- Chord Detection & Reference Table in two columns on larger screens, stacked on smaller screens -->
				<div class="flex flex-col gap-2 sm:flex-row">
					<!-- Left: Chord Detection -->
					<div class="flex-1">
						<SheetMusic notes={activeNotes} signature={currentSignature} />
						<ChordDisplay notes={activeNotes} signature={currentSignature} debug={false} />
					</div>

					<!-- Right: Chord Reference Table -->
					<ChordScaleReference
						signature={currentSignature}
						notes={activeNotes}
						detectedChords={currentChords}
						{matchingChords}
					/>
				</div>
			</div>

			<!-- Always show the piano keyboard -->
			<div class="piano-wrapper">
				<Piano
					notes={activeNotes}
					readonly={false}
					compact={false}
					showLabels={true}
					signature={currentSignature}
					on:notePress={handlePianoNotePress}
				/>
			</div>

			<!-- Keyboard shortcuts info -->
			<div class="mt-1 border-t pt-1 text-center text-xs text-gray-500">
				{#if samplesLoaded}
					<p>
						Keyboard shortcuts: <span class="mx-1 rounded bg-gray-200 px-1 py-0.5 font-mono"
							>Space</span
						>
						to save chord,
						<span class="mx-1 rounded bg-gray-200 px-1 py-0.5 font-mono">P</span> to play most recent
						chord
					</p>
				{:else}
					<div class="flex flex-col items-center">
						<p class="mb-0.5">Loading piano samples: {loadingProgress}%</p>
						<div class="h-1.5 w-full max-w-xs overflow-hidden rounded-full bg-gray-200">
							<div class="h-full rounded-full bg-blue-500" style="width: {loadingProgress}%"></div>
						</div>
					</div>
				{/if}
			</div>
		</div>
	</div>
</div>

<style>
	div::-webkit-scrollbar {
		width: 8px;
	}

	div::-webkit-scrollbar-track {
		background: #f1f1f1;
		border-radius: 10px;
	}

	div::-webkit-scrollbar-thumb {
		background: #d1d5db;
		border-radius: 10px;
	}

	div::-webkit-scrollbar-thumb:hover {
		background: #9ca3af;
	}

	.focus-icon {
		position: fixed;
		bottom: 20px;
		right: 20px;
		width: 40px;
		height: 40px;
		background-color: #3b82f6;
		color: white;
		border-radius: 50%;
		display: flex;
		align-items: center;
		justify-content: center;
		box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
		z-index: 30;
		cursor: pointer;
	}
</style>
