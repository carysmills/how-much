<script>
	import { onDestroy } from "svelte";
	import data from "./data/cost-filtered.json";
	import { selected } from "./stores.js";
	import Sparkline from "./Sparkline.svelte";

	let selectedItems;
	let total2011;
	let total2021;
	let difference;
	let differenceDetails;
	let formattedDifference;

	function percIncrease(a, b) {
		let percent;
		if (b !== 0) {
			if (a !== 0) {
				percent = ((b - a) / a) * 100;
			} else {
				percent = b * 100;
			}
		} else {
			percent = -a * 100;
		}
		return String(Math.floor(percent));
	}

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
		total2011 = getTotalCost($selected, "2011-12");
		total2021 = getTotalCost($selected, "2021-12");
		difference = total2021 - total2011;
		differenceDetails = getDetails(difference);
		formattedDifference = formatCurrency(Math.abs(difference));
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

		return total;
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
		const startAmount = getDateCost(item, "2011-12");
		const endAmount = getDateCost(item, "2021-12");
		const difference = endAmount - startAmount;
		const percentDifference = percIncrease(startAmount, endAmount);
		const { symbol, color } = getDetails(difference);
		return `<span style="color: ${color}"}>${symbol}</span> ${formatCurrency(
			Math.abs(difference)
		)} (${percentDifference}%)`;
	}

	const unsubscribe = selected.subscribe((value) => {
		selectedItems = value;
	});

	onDestroy(unsubscribe);
</script>

<div class="receipt paper">
	<h3>
		Average Canadian grocery prices<br />
		Jan. 2022 compared to Jan. 2012<br />
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
					<span class="indicator">
						{@html getCostDifference(item)}
					</span>
				</div>
				<Sparkline dataItem={data[item]} {item} />
			</li>
		{/each}
	</ul>

	<hr />

	<div class="total">
		<p class="finalTotals">
			TOTAL:
			{formatCurrency(total2021)}
		</p>

		<p class="totalIndicator">
			<span class="indicator"
				>{@html `<span style="color: ${differenceDetails.color}"}>${differenceDetails.symbol}</span> ${formattedDifference}`}</span
			>
		</p>
	</div>
</div>
