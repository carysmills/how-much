<script>
	import data from "./data/cost-filtered.json";
	import { selected } from "./stores.js";

	import { chart } from "svelte-apexcharts";

	let selectedItems;

	function formatTitle(title) {
		const indexOfBracket = title.indexOf("(");
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

	function getDetails(difference) {
		if (difference === 0) {
			return { symbol: "", color: "grey" };
		} else if (difference < 0) {
			return { symbol: "&#9660;", color: "red" };
		} else {
			return { symbol: "&#9650;", color: "green" };
		}
	}

	function getCostDifference(item) {
		const difference =
			getDateCost(item, "2021-12") - getDateCost(item, "2011-12");
		const { symbol, color } = getDetails(difference);
		return `<span style="color: ${color}"}>${symbol}</span> ${formatCurrency(
			Math.abs(difference)
		)}`;
	}

	function updateItem(item) {
		selected.update((n) => {
			const itemExists = n.includes(item);
			if (itemExists) {
				return n.filter((x) => x != item);
			} else {
				return [...n, item].sort();
			}
		});

		totalCostCurrent = getTotalCost("2021-12");
		totalCostPast = getTotalCost("2011-12");
	}
</script>

<div>
	<h1>A decade of grocery prices</h1>

	<p class="intro">
		Wondering what inflation means for your grocery bill? This interactive
		uses <a
			href="https://www150.statcan.gc.ca/t1/tbl1/en/tv.action?pid=1810000201&cubeTimeFrame.startMonth=12&cubeTimeFrame.startYear=2011&cubeTimeFrame.endMonth=12&cubeTimeFrame.endYear=2021&referencePeriods=20111201%2C20211201"
			>Statistics Canada monthly average food price data</a
		> to compare December 2021 prices to those from a decade before. Pick the
		groceries your interested in and see them reflected on the bill.
	</p>

	<main>
		<div class="card">
			<div class="card-header">
				<h2 class="card-title">Select groceries</h2>
			</div>
			{#each Object.keys(data) as item}
				<div class="card-text">
					<label class="itemContainer" for={item}>
						<input
							type="checkbox"
							checked={selectedItems.includes(item)}
							id={item}
							name={item}
							on:change={(item) => {
								updateItem(item.target.name);
							}}
						/>
						{formatTitle(item)}
					</label>
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
						<div class="itemText">
							<strong>{item}</strong>
							<p>
								{formatCurrency(getDateCost(item, "2021-12"))}
							</p>
							<p>
								{@html getCostDifference(item)}
							</p>
						</div>
						<div use:chart={createChartOptions(data[item])} />
					</li>
				{/each}
			</ul>

			<hr />

			<div class="total">
				<strong>TOTALS FOR SELECTED GROCERIES</strong>
				<p>*{totalCostCurrent}: 2021*</p>
				<p>*{totalCostPast}: 2011*</p>
			</div>
		</div>
	</main>
</div>

<style>
	* {
		font-family: "Courier New", Courier, monospace;
	}

	h1 {
		font-size: 60px;
		color: #f16775;
		font-weight: 700;
	}

	h1,
	h3,
	.intro,
	a {
		font-family: "Montserrat", sans-serif;
	}

	.intro {
		max-width: 800px;
		padding-bottom: 60px;
		margin: 0 auto;
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
		font-weight: 500;
	}

	.total {
		text-align: center;
	}

	.itemText {
		width: 200px;
	}

	.paper {
		background: whitesmoke;
		padding: 30px;
		position: relative;
	}

	.paper,
	.paper::before,
	.paper::after {
		box-shadow: rgba(100, 100, 111, 0.1) 0px 7px 20px 0px;
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
		transform: rotate(2deg);
		z-index: -1;
	}

	.paper::after {
		top: 5px;
		right: -5px;
		transform: rotate(-2deg);
		z-index: -2;
	}

	.card {
		transform: rotate(-0.002turn);
		background-color: white;
		background: repeating-linear-gradient(
			white,
			white 25px,
			#9198e5 26px,
			#9198e5 27px
		);
		background-position-y: 34px;
		padding: 0;
		margin-right: 15px;
		box-shadow: rgba(100, 100, 111, 0.1) 0px 7px 20px 0px;
		border: 1px solid whitesmoke;
		padding-bottom: 60px;
		height: max-content;
	}
	.card-header {
		background: linear-gradient(white, white 33px, pink 35px, pink 36px);
		height: 36px;
	}
	.card-title {
		position: relative;
		left: 10px;
		top: 2px;
		font-size: 30px;
	}
	.card-text {
		position: relative;
		top: 30px;
		font-size: 18px;
		margin: 0 20px;
		line-height: 27px;
	}

	a {
		color: #373e98;
	}

	@media only screen and (max-width: 768px) {
		main {
			flex-direction: column;
		}

		.receipt {
			margin-top: 30px;
		}
	}
</style>
