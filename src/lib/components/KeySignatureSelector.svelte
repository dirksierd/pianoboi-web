<!-- Component for selecting key signatures -->
<script lang="ts">
	import type { Signature } from '$lib/types/signatures';
	import { signatures } from '$lib/types/signatures';
	import { createEventDispatcher } from 'svelte';

	export let currentSignature: Signature;

	const dispatch = createEventDispatcher<{
		select: { signature: Signature };
	}>();

	let isKeyMenuOpen = false;

	function toggleKeyMenu() {
		isKeyMenuOpen = !isKeyMenuOpen;
	}

	function selectSignature(signature: Signature) {
		dispatch('select', { signature });
		isKeyMenuOpen = false;
	}
</script>

<div class="relative">
	<button
		class="flex items-center gap-1 rounded-lg border bg-white px-2 py-1 text-xs font-medium shadow-sm hover:bg-gray-50"
		on:click={toggleKeyMenu}
	>
		<!-- Music Note Icon -->
		<svg
			xmlns="http://www.w3.org/2000/svg"
			class="h-3.5 w-3.5 text-blue-600"
			viewBox="0 0 20 20"
			fill="currentColor"
		>
			<path
				d="M18 3a1 1 0 00-1.196-.98l-10 2A1 1 0 006 5v9.114A4.369 4.369 0 005 14c-1.657 0-3 .895-3 2s1.343 2 3 2 3-.895 3-2V7.82l8-1.6v5.894A4.37 4.37 0 0015 12c-1.657 0-3 .895-3 2s1.343 2 3 2 3-.895 3-2V3z"
			/>
		</svg>
		<span class="font-medium text-gray-700">Key:</span>
		<span class="ml-1 text-blue-600">{currentSignature.id}</span>
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

	{#if isKeyMenuOpen}
		<div
			class="absolute left-0 top-full z-30 mt-1 w-72 rounded-md border border-gray-200 bg-white p-2 shadow-lg"
		>
			<div class="mb-1 border-b pb-1 text-xs font-medium text-gray-700">Key Signatures</div>
			<div class="grid grid-cols-2 gap-1">
				{#each signatures as sig}
					<button
						class="flex flex-col items-start rounded-md px-2 py-1 text-left text-xs transition-colors hover:bg-blue-50"
						class:bg-blue-100={currentSignature === sig}
						on:click={() => selectSignature(sig)}
					>
						<span class="font-medium">{sig.id}</span>
						<span class="text-[10px] text-gray-500">{sig.label}</span>
						{#if sig.sharps > 0}
							<span
								class="mt-0.5 inline-flex items-center rounded-md bg-amber-50 px-1 py-0.5 text-[10px] font-medium text-amber-700"
							>
								{sig.sharps}
								{sig.sharps === 1 ? 'sharp' : 'sharps'}
							</span>
						{:else if sig.flats > 0}
							<span
								class="mt-0.5 inline-flex items-center rounded-md bg-sky-50 px-1 py-0.5 text-[10px] font-medium text-sky-700"
							>
								{sig.flats}
								{sig.flats === 1 ? 'flat' : 'flats'}
							</span>
						{:else}
							<span
								class="mt-0.5 inline-flex items-center rounded-md bg-gray-50 px-1 py-0.5 text-[10px] font-medium text-gray-600"
							>
								No sharps/flats
							</span>
						{/if}
					</button>
				{/each}
			</div>
		</div>
	{/if}
</div>
