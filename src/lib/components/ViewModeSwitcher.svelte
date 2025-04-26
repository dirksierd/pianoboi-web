<!-- Component for switching between keyboard and sheet view modes -->
<script lang="ts">
	import { createEventDispatcher } from 'svelte';

	export let viewMode: 'keyboard' | 'sheet' = 'keyboard';

	const dispatch = createEventDispatcher<{
		change: { mode: 'keyboard' | 'sheet' };
	}>();

	function toggleViewMode(mode: 'keyboard' | 'sheet') {
		dispatch('change', { mode });
	}
</script>

<div class="flex overflow-hidden rounded-md border shadow-sm">
	<button
		class="flex-1 px-2 py-1 text-sm font-medium transition-colors"
		class:bg-blue-500={viewMode === 'keyboard'}
		class:text-white={viewMode === 'keyboard'}
		class:bg-white={viewMode !== 'keyboard'}
		class:text-gray-700={viewMode !== 'keyboard'}
		on:click={() => toggleViewMode('keyboard')}
	>
		<div class="flex items-center justify-center gap-1">
			<svg
				xmlns="http://www.w3.org/2000/svg"
				class="h-3.5 w-3.5"
				viewBox="0 0 20 20"
				fill="currentColor"
			>
				<path d="M13 7H7v6h6V7z" />
				<path
					fill-rule="evenodd"
					d="M7 2a1 1 0 00-1 1v1H5a3 3 0 00-3 3v10a3 3 0 003 3h10a3 3 0 003-3V7a3 3 0 00-3-3h-1V3a1 1 0 00-1-1H7zm1 3V4h4v1H8z"
					clip-rule="evenodd"
				/>
			</svg>
			<span class="hidden sm:inline text-xs">Keyboard</span>
		</div>
	</button>
	<button
		class="flex-1 px-2 py-1 text-sm font-medium transition-colors"
		class:bg-blue-500={viewMode === 'sheet'}
		class:text-white={viewMode === 'sheet'}
		class:bg-white={viewMode !== 'sheet'}
		class:text-gray-700={viewMode !== 'sheet'}
		on:click={() => toggleViewMode('sheet')}
	>
		<div class="flex items-center justify-center gap-1">
			<svg
				xmlns="http://www.w3.org/2000/svg"
				class="h-3.5 w-3.5"
				viewBox="0 0 20 20"
				fill="currentColor"
			>
				<path
					fill-rule="evenodd"
					d="M4 4a2 2 0 012-2h8a2 2 0 012 2v12a2 2 0 01-2 2H6a2 2 0 01-2-2V4zm2-1a1 1 0 00-1 1v12a1 1 0 001 1h8a1 1 0 001-1V4a1 1 0 00-1-1H6z"
					clip-rule="evenodd"
				/>
				<path d="M7 5h6v2H7V5zm0 4h6v2H7V9zm0 4h6v2H7v-2z" />
			</svg>
			<span class="hidden sm:inline text-xs">Sheet</span>
		</div>
	</button>
</div>
