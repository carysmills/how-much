<script>
	import { onDestroy } from "svelte";
	import data from "./data/cost-filtered.json";
	import { selected } from "./stores.js";
	import Sparkline from "./Sparkline.svelte";

	let selectedItems;

	function formatCurrency(number) {
		return new Intl.NumberFormat("en", {
			style: "currency",
			currency: "USD",
		}).format(number);
	}

	function getDateCost(item, date) {
		return data[item].filter((data) => data.date === date)[0].cost;
	}

	selected.subscribe((value) => {
		selectedItems = value;
	});

	function getTotalCost(value, currentDate) {
		if (value == null || value.length === 0) {
			return 0;
		}

		const total = value.reduce((previousValue, currentValue) => {
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

	const unsubscribe = selected.subscribe((value) => {
		selectedItems = value;
	});

	onDestroy(unsubscribe);
</script>

<div class="receipt paper">
	<h3>
		Average Canadian grocery prices<br />
		Dec. 2021 compared to Dec. 2011<br />
		🛒
	</h3>

	<hr />
	<ul>
		{#each $selected as item (item)}
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
				<Sparkline dataItem={data[item]} />
			</li>
		{/each}
	</ul>

	<hr />

	<div class="total">
		<strong>TOTALS FOR SELECTED GROCERIES</strong>
		<p>*{getTotalCost($selected, "2021-12")}: 2021*</p>
		<p>*{getTotalCost($selected, "2011-12")}: 2011*</p>
	</div>
</div>

<style>
	* {
		font-family: "Courier New", Courier, monospace;
	}

	h3 {
		font-family: "Montserrat", sans-serif;
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
		animation-duration: 0, 0.5s;
		text-transform: uppercase;
	}

	.receipt {
		height: max-content;
		padding: 20px;
		font-family: "Courier New", Courier, monospace;
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

	:root {
		--form-control-color: #373e98;
	}

	@media only screen and (max-width: 768px) {
		.receipt {
			margin-top: 30px;
		}
	}
</style>
