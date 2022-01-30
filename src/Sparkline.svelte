<script>
	import { chart } from "svelte-apexcharts";

	export let dataItem = [];

	function createChartOptions(data) {
		const [cost, date] = data.reduce(
			([a, b], { cost, date }) => {
				a.push(cost);
				b.push(date);
				return [a, b];
			},
			[[], []]
		);

		return {
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
					name: "sales",
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
	}
</script>

<div use:chart={createChartOptions(dataItem)} />

