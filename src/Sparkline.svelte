<script>
	import { chart } from "svelte-apexcharts";

	export let dataItem = [];
	export let item = "";

	const [cost, date] = dataItem.reduce(
		([a, b], { cost, date }) => {
			a.push(cost);
			b.push(date);
			return [a, b];
		},
		[[], []]
	);

	const options = {
		chart: {
			type: "area",
			toolbar: { show: false },
			width: 150,
			height: 75,
		},
		fill: {
			colors: ["#373e98"],
		},
		legend: { show: false, height: 0, width: 0 },
		tooltip: { enabled: false },
		dataLabels: { enabled: false },
		grid: { show: false },
		yaxis: { show: false, axisTicks: { show: false }, min: 0 },
		sparkline: {
			enabled: true,
		},
		stroke: { width: 1, colors: ["#373e98"] },
		series: [
			{
				name: "cost",
				data: cost,
			},
		],
		xaxis: {
			axisTicks: { show: false },
			show: false,
			categories: date,
			tickAmount: 0,
			labels: { show: false, maxHeight: 0 },
		},
	};
</script>

<div
	role="figure"
	aria-label="A sparkline showing the sales of {item}"
	aria-describedby={`table-${item}`}
	use:chart={options}
/>

<table id={`table-${item}`}>
	<tr>
		<th>Date</th>
		<th>Cost</th>
	</tr>

	{#each dataItem as row (row)}
		<tr>
			<td>{row.date}</td>
			<td>{row.cost}</td>
		</tr>
	{/each}
</table>

<style>
	table {
		position: absolute;
		left: -10000px;
		top: auto;
		width: 1px;
		height: 1px;
		overflow: hidden;
	}
</style>
