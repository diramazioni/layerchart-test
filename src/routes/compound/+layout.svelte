<script lang="ts">
	import { setContext } from 'svelte';
	import allData from './data.ts';

	let { children } = $props();

	type DataItem = {
		date: Date;
		rain: number;
		cumulative: number;
	};

	type FilterType = 'from-oct-11' | 'oct-1-only' | 'sep-30-to-oct-11';

	let selectedFilter = $state<FilterType>('from-oct-11');

	let filteredData = $derived(() => {
		const startDate2025Oct11 = new Date('2025-10-11T21:00:00.000Z');
		const startDate2025Sep30 = new Date('2025-09-30T22:00:00.000Z');
		const endDate2025Oct1 = new Date('2025-10-01T22:00:00.000Z');

		switch (selectedFilter) {
			case 'from-oct-11':
				return allData.filter(d => d.date >= startDate2025Oct11);
			case 'oct-1-only':
				return allData.filter(d => d.date >= startDate2025Sep30 && d.date <= endDate2025Oct1);
			case 'sep-30-to-oct-11':
				return allData.filter(d => d.date >= startDate2025Sep30 && d.date <= startDate2025Oct11);
			default:
				return allData;
		}
	});

	let filterDescription = $derived(() => {
		switch (selectedFilter) {
			case 'from-oct-11':
				return 'One single non 0 data point for October 11, 2025';
			case 'oct-1-only':
				return 'Several data points: The scale for rain is not correctly scaled to the rainDomain';
			case 'sep-30-to-oct-11':
				return 'This cause error for Error: <line> attribute x2: Expected length, "NaN". !!!  open the console:  The x axis fails to show several date ticks';
			default:
				return 'Tutti i dati';
		}
	});

	setContext<{ data: DataItem[] }>('filteredData', {
		get data() {
			return filteredData();
		}
	});
</script>

<div class="p-4">
	<div class="mb-4 space-y-3">
		<div class="flex gap-2 items-center">
			<a href="/" class="text-blue-500 hover:text-blue-900">Home</a>
			<span class="text-gray-400">|</span>
			<div class="flex gap-2">
				<button
					onclick={() => selectedFilter = 'from-oct-11'}
					class="px-4 py-2 rounded {selectedFilter === 'from-oct-11' ? 'bg-blue-500 text-white' : 'bg-gray-200 text-gray-700'}"
				>
					Dal 11 Ott
				</button>
				<button
					onclick={() => selectedFilter = 'oct-1-only'}
					class="px-4 py-2 rounded {selectedFilter === 'oct-1-only' ? 'bg-blue-500 text-white' : 'bg-gray-200 text-gray-700'}"
				>
					1 Ott
				</button>
				<button
					onclick={() => selectedFilter = 'sep-30-to-oct-11'}
					class="px-4 py-2 rounded {selectedFilter === 'sep-30-to-oct-11' ? 'bg-blue-500 text-white' : 'bg-gray-200 text-gray-700'}"
				>
					30 Set - 11 Ott
				</button>
			</div>
		</div>
		<div class="bg-blue-50 border border-blue-200 rounded-md px-4 py-2 text-sm text-blue-800">
			{filterDescription()} ({filteredData().length} data points)
		</div>
	</div>
	{@render children()}
</div>
