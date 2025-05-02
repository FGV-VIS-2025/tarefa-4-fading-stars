<script>
	import consMap from "$lib/data/constellations-map.json";
	import * as d3 from "d3";
	export let starsRaw; //Input of all stars, this selector never changes so it needs to be made with all the data
	export let selectedCons; //Output of the component - the filtered cons
	export let consPosition; //Output of the component - the selected cons location

	//To hold what the user types in input
	let userQuery = "";
	//Initial constellation list
	let constellationList = d3
		.rollups(
			starsRaw,
			(v) => v.length,
			(d) => d.con,
		)
		.map(([cons, appearances]) => cons)
		.sort();
	//object to mark whenever a constellation is checked
	let checkedCons = {};
	//map of constellation -> angle to focus
	let consAngle = {};
	function consFocus(cons) {
		//star with lowest magnitude
		let stars = starsRaw.filter((star) => star.con == cons);
		let x_proj =
			stars.map((star) => star.x_proj).reduce((a, b) => a + b) /
			stars.length;
		let y_proj =
			stars.map((star) => star.y_proj).reduce((a, b) => a + b) /
			stars.length;
		let z_proj =
			stars.map((star) => star.z_proj).reduce((a, b) => a + b) /
			stars.length;
		let length = Math.sqrt(x_proj ** 2 + y_proj ** 2 + z_proj ** 2);
		x_proj /= length;
		y_proj /= length;
		z_proj /= length;
		let X = Math.asin(y_proj);
		let Y = Math.atan2(x_proj, z_proj);
		return { X: X, Y: Y };
	}

	for (let cons of constellationList) {
		checkedCons[cons] = true;
		consAngle[cons] = consFocus(cons);
	}

	function setVisibility(bool) {
		for (let cons of constellationList) {
			checkedCons[cons] = bool;
		}
	}

	let queriedCons;
	$: queriedCons = constellationList.filter(
		(cons) =>
			cons != null &&
			consMap[cons].toLowerCase().includes(userQuery.toLowerCase()),
	);

	$: selectedCons = Object.keys(checkedCons).filter(
		(key) => checkedCons[key],
	);
</script>

<div class="container">
	<h3>Filtre as constelações visíveis.</h3>
	<div class="searchBox">
		<label for="consSearch">Busque por uma constelação!</label>
		<input type="text" id="consSearch" bind:value={userQuery} />
		<div class="constellations">
			{#each queriedCons as cons, index}
				<div class="constellation">
					<a
						class="showButton"
						on:click={(evt) => (consPosition = consAngle[cons])}
						>Focar
					</a>
					<label class="consName">
						{consMap[cons]}
						<input
							name="marker{cons}"
							class="showCheck"
							type="checkbox"
							bind:checked={checkedCons[cons]}
						/>
					</label>
				</div>
			{/each}
		</div>
	</div>
	{#if !constellationList.map((cons) => checkedCons[cons]).every((b) => b)}
		<button type="button" on:click={(evt) => setVisibility(true)}
			>Mostrar todas</button
		>
	{:else}
		<button type="button" on:click={(evt) => setVisibility(false)}
			>Ocultar todas</button
		>
	{/if}
</div>

<style>
	.container {
		border-style: solid;
		border-radius: 6px;
		border-width: 2px;

		margin-bottom: 20px;

		padding: 20px;
	}

	h3 {
		margin: auto;
		text-align: justify;
	}

	.searchBox {
		position: relative;
		width: 100%;
		margin-bottom: 1ch;
	}

	#consSearch {
		width: 100%;
		box-sizing: border-box;
	}

	.constellations {
		display: none;
		position: absolute;
		top: 100%;
		left: 0;
		right: 0;

		margin: 0;
		padding: 1ch;

		background-color: var(--accent-black);
		backdrop-filter: 20px;
		border: 2px solid var(--border-color);
		border-radius: 12px;

		flex-direction: column;

		max-height: 15ch;
		width: auto;
		overflow-y: auto;
	}

	.constellation {
		margin: 0.5ch 0;
	}

	.showCheck {
		display: none;
	}

	.searchBox:hover .constellations {
		display: flex;
	}

	.searchBox:focus-within .constellations {
		display: flex;
	}

	.constellation:hover .showCheck {
		display: inline;
	}

	/* .consName{
    color: black;
} */

	.showButton {
		cursor: pointer;
		color: #b4b4b4;
		font-size: 80%;
		text-decoration: underline;
	}
</style>
