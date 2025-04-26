<!-- Component for viewing chord scale reference -->
<script lang="ts">
	import type { Signature } from '$lib/types/signatures';
	import { generateScaleChords, isInKey } from '$lib/utils/chords';

	export let signature: Signature;
	export let notes: any[] = [];
	export let detectedChords: string[] = [];
	export let matchingChords = { majorMatches: [], minorMatches: [] };
	export let hasInfoButton = true;

	// Scale degrees
	$: scaleChords = generateScaleChords(signature);

	let isInfoOpen = false;
	function toggleInfo() {
		isInfoOpen = !isInfoOpen;
	}
</script>

<div class="flex w-full flex-1 flex-col rounded-lg border bg-white p-3 shadow-sm">
	<div class="mb-4 flex items-center justify-between border-b pb-2">
		<h3 class="text-sm font-medium text-gray-700">
			Key: {signature.label}
		</h3>

		<!-- Info Button -->
		{#if hasInfoButton}
			<div class="relative">
				<button
					class="ml-2 rounded-full p-1 text-gray-400 hover:bg-gray-100 hover:text-gray-600"
					on:click={toggleInfo}
					title="Show chord notation info"
				>
					<svg
						xmlns="http://www.w3.org/2000/svg"
						class="h-4 w-4"
						viewBox="0 0 20 20"
						fill="currentColor"
					>
						<path
							fill-rule="evenodd"
							d="M18 10a8 8 0 11-16 0 8 8 0 0116 0zm-7-4a1 1 0 11-2 0 1 1 0 012 0zM9 9a1 1 0 000 2v3a1 1 0 001 1h1a1 1 0 100-2v-3a1 1 0 00-1-1H9z"
							clip-rule="evenodd"
						/>
					</svg>
				</button>

				{#if isInfoOpen}
					<div
						class="absolute right-0 top-full z-40 mt-2 w-72 rounded-lg border border-gray-200 bg-white p-3 shadow-lg"
					>
						<div class="space-y-2 text-xs text-gray-500">
							<p>
								<span class="font-medium">Major Scale:</span> The highlighted chords are the primary
								chords:
								<span class="font-mono">I</span> (tonic),
								<span class="font-mono">IV</span> (subdominant), and
								<span class="font-mono">V</span> (dominant).
							</p>
							<p>
								Uppercase numerals (I, IV, V) indicate major chords, while lowercase (ii, iii, vi)
								indicate minor chords. The diminished symbol (°) in vii° indicates a diminished
								chord.
							</p>
							<p>
								<span class="font-medium">Minor Scale:</span> The highlighted chords are the primary
								chords:
								<span class="font-mono">i</span> (tonic),
								<span class="font-mono">iv</span> (subdominant), and
								<span class="font-mono">v</span> (dominant).
							</p>
							<p>
								Lowercase numerals (i, ii°, iv, v) indicate minor chords, while uppercase (III, VI,
								VII) indicate major chords. The diminished symbol (°) in ii° indicates a diminished
								chord.
							</p>
						</div>
					</div>
				{/if}
			</div>
		{/if}
	</div>

	<!-- Detected Chords Display -->
	{#if notes && notes.length > 0 && detectedChords && detectedChords.length > 0}
		<div class="mb-4">
			<h4 class="mb-2 text-sm font-medium text-gray-700">Detected Chords</h4>
			<div class="flex flex-wrap gap-2">
				{#each detectedChords as chord}
					<span
						class="rounded-full px-3 py-1 text-sm font-medium"
						class:bg-green-100={isInKey(chord, scaleChords)}
						class:text-green-800={isInKey(chord, scaleChords)}
						class:bg-blue-100={!isInKey(chord, scaleChords)}
						class:text-blue-800={!isInKey(chord, scaleChords)}
					>
						{chord}
						{#if isInKey(chord, scaleChords)}
							<span class="ml-1 text-xs text-green-600">(in key)</span>
						{/if}
					</span>
				{/each}
			</div>
		</div>
	{/if}

	<div class="overflow-hidden rounded border">
		<table class="w-full text-sm">
			<thead class="bg-gray-50 text-xs font-medium text-gray-700">
				<tr>
					<th class="py-1 pl-2 text-left">Scale</th>
					<th class="py-1 text-center">I</th>
					<th class="py-1 text-center">II</th>
					<th class="py-1 text-center">III</th>
					<th class="py-1 text-center">IV</th>
					<th class="py-1 text-center">V</th>
					<th class="py-1 text-center">VI</th>
					<th class="py-1 text-center">VII</th>
				</tr>
			</thead>
			<tbody>
				<tr class="border-t">
					<td class="bg-blue-50 px-2 py-1 text-xs font-medium">Major</td>
					{#each scaleChords.major as chord, i}
						<td
							class="py-1 text-center text-xs"
							class:bg-blue-100={i === 0 || i === 3 || i === 4}
							class:bg-green-200={matchingChords.majorMatches.includes(i)}
							class:font-bold={matchingChords.majorMatches.includes(i)}
						>
							<div>{chord}</div>
							<div class="mt-0.5 text-[10px] text-gray-500">
								{i === 0
									? 'I'
									: i === 1
										? 'ii'
										: i === 2
											? 'iii'
											: i === 3
												? 'IV'
												: i === 4
													? 'V'
													: i === 5
														? 'vi'
														: 'vii°'}
							</div>
						</td>
					{/each}
				</tr>
				<tr class="border-t">
					<td class="bg-blue-50 px-2 py-1 text-xs font-medium">Minor</td>
					{#each scaleChords.minor as chord, i}
						<td
							class="py-1 text-center text-xs"
							class:bg-blue-100={i === 0 || i === 3 || i === 4}
							class:bg-green-200={matchingChords.minorMatches.includes(i)}
							class:font-bold={matchingChords.minorMatches.includes(i)}
						>
							<div>{chord}</div>
							<div class="mt-0.5 text-[10px] text-gray-500">
								{i === 0
									? 'i'
									: i === 1
										? 'ii°'
										: i === 2
											? 'III'
											: i === 3
												? 'iv'
												: i === 4
													? 'v'
													: i === 5
														? 'VI'
														: 'VII'}
							</div>
						</td>
					{/each}
				</tr>
			</tbody>
		</table>
	</div>
</div>
