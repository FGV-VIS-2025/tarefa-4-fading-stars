<script>
	import consMap from "$lib/data/constellations-map.json";

	import Help from "$lib/components/Help.svelte";

	import * as d3 from "d3";
	export let starsRaw; //Input of all stars, this selector never changes so it needs to be made with all the data
	export let selectedCons; //Output of the component - the filtered cons
	export let consPosition = { position: { X: 0, Y: 0 }, name: "" }; //Output of the component - the selected cons location

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
		return { position: { X: X, Y: Y }, name: cons };
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
	<div class="search-box">
		<label for="cons-search">Busque por uma constelação!</label>
		<input type="text" id="cons-search" bind:value={userQuery} />
		<div class="constellations">
			{#each queriedCons as cons, index}
				<div class="constellation">
					<button class="cons-name"
							on:click={() => (consPosition = consAngle[cons])}
							style="all:unset; cursor:pointer;">
						<span class="focus-button">Focar:</span>
						{consMap[cons]}
					</button>

					<input
						alt=""
						name="marker{cons}"
						type="image"
						src="{checkedCons[cons]? "eye-regular.svg" : "eye-slash-regular.svg"}"
						class="show-button"
						on:click={() => checkedCons[cons] = !checkedCons[cons]}
					/>
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

	<Help>
		<p>
			As constelações foram essenciais para a história humana. Cada uma
			das 88 constelações definidas atualmente é relacionada com
			diferentes mitos e lendas, e seus papéis foram importantes para o
			desenvolvimento humano: constelações como Ursa Maior e o Cruzeiro do
			Sul, por exemplo, permitiam com que humanos se localizassem e
			guiassem suas viagens sem a necessidade de GPS.
		</p>
		<p>
			Por meio desse filtro, é possível tanto selecionar quais
			constelações aparecem na visualização do globo celeste quanto focar
			em alguma específica. Quando o foco é aplicado, o globo se move para
			simular um observador que tem a constelação no topo do céu, e ela
			brilha momentaneamente para sua localização fique clara. é possível
			filtrar a lista pelo nome de uma constelação específica, em inglês.
		</p>
	</Help>
</div>

<style>
	.container {
		border-style: solid;
		border-radius: 6px;
		border-width: 2px;

		margin-bottom: 20px;

		padding: 2ch;
		position: relative;
	}

	h3 {
		margin: auto;
		text-align: justify;
	}

	.search-box {
		position: relative;
		width: 100%;
		margin-bottom: 1ch;
	}

	#cons-search {
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
		border: 2px solid var(--border-color);
		border-radius: 12px;

		flex-direction: column;

		max-height: 15ch;
		width: auto;
		overflow-y: auto;
		z-index: 2;
	}

	.constellation {
		margin: 0.5ch 0;
		display: flex;
		justify-content: space-between;
	}

	.search-box:hover .constellations {
		display: flex;
	}

	.search-box:focus-within .constellations {
		display: flex;
	}

	.cons-name {
		cursor: pointer;
	}

	.focus-button {
		color: #b4b4b4;
		font-size: 80%;
		text-decoration: underline;
	}

	.show-button {
		margin: 0 2ch;
		height: 1.1em;
		filter: invert(100%);
	}
</style>
