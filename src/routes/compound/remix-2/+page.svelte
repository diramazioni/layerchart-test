<script lang="ts">
	import { Axis, Bars, BarChart, LineChart, Group, Tooltip, Spline } from 'layerchart';
	import { getContext } from 'svelte';

	import { scaleLinear, scaleTime } from 'd3-scale';
	import { extent, max, nice } from 'd3-array';
	import { timeDay } from 'd3-time';

	type DataItem = {
		date: Date;
		rain: number;
		cumulative: number;
	};

	let dataContext = getContext<{ data: DataItem[] }>('filteredData');
	let data = $derived(dataContext.data);

	// map `rain` values to `cumulative` scales values
	let maxRain = $derived(max(data, (d) => d.rain) || 0);
	let maxCumulative = $derived(max(data, (d) => d.cumulative) || 0);
	let rainDomain = $derived(maxRain > 0 ? nice(0, maxRain, 5) : [0, 1]);
	let cumulativeDomain = $derived(maxCumulative > 0 ? [0, maxCumulative] : [0, 1]);
	let baselineScale = $derived(
		scaleLinear(
			cumulativeDomain,
			rainDomain,
		)
	);
</script>

<h2>Inverting the mapping, map cumulativeDomain (big) to rainDomain (small)</h2>
<h3>I know this is <strong>wrong</strong> but in this way the cumulative values are rendered correctly in the chart, (however the rain values are now not scaled correctly)</h3>
<div class="container">
	<BarChart
		{data}
		x="date"
		series={[
			{
				key: 'cumulative',
				color: 'oklch(0.6733 0.272 22.24)',
				value: (d) => baselineScale(d.cumulative)
			},
			{
				key: 'rain',
				color: 'oklch(0.6733 0.1258 207.53)',
			}
		]}
		padding={{ top: 24, bottom: 48, left: 24, right: 32 }}
		legend
	>
		{#snippet marks({ context, visibleSeries })}
			{#each visibleSeries as s, i}
				{#if s.key === 'cumulative'}
					<Spline y={s.value} color={s.color} />
				{:else}
					<Bars y={s.value} insets={{ x: 1 }} color={s.color} />
				{/if}
			{/each}
		{/snippet}

		{#snippet axis({ context, visibleSeries })}
			{@const visibleSeriesKeys = visibleSeries.map((s) => s.key)}

			<!-- Custom bottom axis not needed once Spline applies xInterval offset -->
			<Axis
				placement="bottom"
				format="time"
				rule
			/>

			{#if visibleSeriesKeys.includes('rain')}
				<Axis
					placement="left"
					label="rain (mm)"
					labelPlacement="start"
					format="metric"
					rule
				/>
			{/if}

			{#if visibleSeriesKeys.includes('cumulative')}
				<Axis
					placement="right"
					label="cumulative (mm)"
					ticks={baselineScale.ticks()}
					scale={scaleLinear(baselineScale.domain(), [context.height, 0])}
					rule
				/>
			{/if}
		{/snippet}

		{#snippet tooltip({ context, visibleSeries })}
			{@const visibleSeriesKeys = visibleSeries.map((s) => s.key)}
			<Tooltip.Root {context}>
				{#snippet children({ data })}
					<Tooltip.Header>{data.date.toLocaleString()}</Tooltip.Header>
					<Tooltip.List>
						{#each visibleSeries as s}
							{@const format = (v: number) => v + ' mm'}
							<Tooltip.Item label={s.key} color={s.color} value={data[s.key as keyof DataItem]} {format} />
						{/each}
					</Tooltip.List>
				{/snippet}
			</Tooltip.Root>
		{/snippet}
	</BarChart>
</div>

<style>
	.container {
		height: 400px;
		margin: 30px;
	}
</style>
