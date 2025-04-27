<script>
	//Components
	import StarsPlot from "$lib/components/stars-plot.svelte";
	import LocationFinder from "$lib/components/get-location.svelte";
	import MagnitudeFilter from "$lib/components/filter-magnitude.svelte";
	import ConstelationFilter from "$lib/components/filter-constelations.svelte"
	import * as d3 from "d3";
	//Data
	import starsRaw from "$lib/data/stars-data.json";
	import linesRaw from "$lib/data/lines-data.json";

	//Magnitude Filter
	let maxMagnitude = 30;
	let startsFiltered = [];
	$: starsFiltered = starsRaw.filter(star => star.mag <= maxMagnitude);

	//Constelation Filter
	let selectedCons = [];
	let linesFiltered = {};
	$: linesFiltered = linesRaw.filter(
		([A, B]) => selectedCons.includes(A.con) && selectedCons.includes(B.con)
	)

	//Place finder
	let userCoordinates = {lat: 0, lon: 0};

</script>

<div class="all">
	<div class="filters">
		<h2>Aqui entram os filtros</h2>
		<LocationFinder bind:coordinates = {userCoordinates}/>
		<p>Você está em {userCoordinates.lat}, {userCoordinates.lon}</p>
		<MagnitudeFilter starsRaw = {starsRaw} bind:maxMagnitude = {maxMagnitude}/>
		<ConstelationFilter starsRaw = {starsRaw} bind:selectedCons = {selectedCons}/>
	</div>

	<div class="viz">
		<h2>Aqui entra a viz</h2>
		<StarsPlot starsRaw = {starsFiltered} linesRaw = {linesFiltered}/>
	</div>
</div>

<style>
	.all {
		display: grid;
		grid-template-columns: 30% 70%;
	}
</style>
