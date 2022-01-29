<script>
	import data from "./data/cost-filtered.json";
	import { selected } from "./stores.js";

	import { chart } from "svelte-apexcharts";

	let selectedItems;

	function formatTitle(title) {
		const indexOfBracket = title.indexOf('(');
		return title.slice(0, indexOfBracket);
	}

	function formatCurrency(number) {
		return new Intl.NumberFormat("en", {
			style: "currency",
			currency: "USD",
		}).format(number);
	}

	function createChartOptions(dataItem) {
		const [cost, date] = dataItem.reduce(
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
				height: 80,
			},
			legend: { show: false, height: 0, width: 0 },
			tooltip: { enabled: false },
			dataLabels: { enabled: false },
			grid: { show: false },
			yaxis: { show: false, axisTicks: { show: false }, min: 0, max: 43 },
			sparkline: {
				enabled: true,
			},
			stroke: { width: 1 },
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

	function getDateCost(item, date) {
		return data[item].filter((data) => data.date === date)[0].cost;
	}

	selected.subscribe((value) => {
		selectedItems = value;
	});

	let totalCostCurrent = getTotalCost("2021-12");
	let totalCostPast = getTotalCost("2011-12");

	function getTotalCost(currentDate) {
		if (selectedItems == null || selectedItems.length === 0) {
			return 0;
		}

		const total = selectedItems.reduce((previousValue, currentValue) => {
			return (
				previousValue +
				Number(
					data[currentValue].filter(
						({ date }) => date == currentDate
					)[0].cost
				)
			);
		}, 0);

		return formatCurrency(total);
	}

	function updateItem(item) {
		selected.update((n) => {
			const itemExists = n.includes(item);
			if (itemExists) {
				return n.filter((x) => x != item);
			} else {
				return [...n, item];
			}
		});

		totalCostCurrent = getTotalCost("2021-12");
		totalCostPast = getTotalCost("2011-12");
	}
</script>

<div>
	<h1>The last decade's grocery prices</h1>

	<main>
		<div>
			<h2>Pick your groceries</h2>
			{#each Object.keys(data) as item}
				<div>
					<input
						type="checkbox"
						checked={selectedItems.includes(item)}
						id={item}
						name={item}
						on:change={(item) => {
							updateItem(item.target.name);
						}}
					/>
					<label class="itemContainer" for={item}>{formatTitle(item)}</label>
				</div>
			{/each}
		</div>

		<div class="receipt paper">
			<h3>
				Average Canadian grocery prices<br />
				Dec. 2021 compared to Dec. 2011<br />
				🛒
			</h3>

			<hr />
			<ul>
				{#each selectedItems as item}
					<li>
						<div>
							<strong>{item}</strong>
							<p>
								{formatCurrency(getDateCost(item, "2021-12"))}
							</p>
							<!-- make up dynamic -->
							<p>
								🔼 {formatCurrency(
									getDateCost(item, "2021-12") -
										getDateCost(item, "2011-12")
								)}
							</p>
						</div>
						<div use:chart={createChartOptions(data[item])} />
					</li>
				{/each}
			</ul>

			<hr />

			<div class="total">
				<p>*{totalCostCurrent}: 2021 total*</p>
				<p>*{totalCostPast}: 2011 total*</p>
			</div>
		</div>
	</main>
</div>

<style>
	* {
		font-family: "Courier New", Courier, monospace;
	}

	h1 {
		text-align: center;
		margin: 40px 0;
	}
	main {
		display: flex;
		justify-content: space-around;
	}

	.itemContainer {
		display: inline-block;
		/* font-family: "Architects Daughter", cursive; */
	}

	ul {
		margin: 0;
		padding: 0;
	}

	@keyframes fadeInOpacity {
		0% {
			opacity: 0;
		}
		100% {
			opacity: 1;
		}
	}

	li {
		display: flex;
		align-items: end;
		justify-content: space-between;
		opacity: 1;
		animation-name: fadeInOpacity;
		animation-iteration-count: 1;
		animation-timing-function: ease-in;
		animation-duration: 0, 5s;
		text-transform: uppercase;
	}

	.receipt {
		height: max-content;
		max-width: 360px;
		transform: rotate(0.005turn);
		padding: 20px;
	}

	.receipt p {
		margin: 0;
	}

	hr {
		border: none;
		border-top: 3px dotted #333;
		margin: 20px;
	}

	h3 {
		text-align: center;
	}

	.total {
		text-align: center;
	}

	.paper {
		background: whitesmoke;
		padding: 30px;
		position: relative;
	}

.paper,
.paper::before,
.paper::after {
  box-shadow: 1px 1px 1px rgba(0,0,0,0.25);
  border: 1px solid whitesmoke;
}

.paper::before,
.paper::after {
  content: "";
  position: absolute;
  height: 100%;
  width: 99%;
  background-color: whitesmoke;
}

.paper::before {
  right: 15px;
  top: 0;
  transform: rotate(-1deg);
  z-index: -1;
}

.paper::after {
  top: 5px;
  right: -5px;
  transform: rotate(1deg);
  z-index: -2;
}
</style>
