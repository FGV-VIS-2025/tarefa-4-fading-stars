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

	let customAngle = {X: 0, Y: 0};

	//Magnitude Filter
	let maxMagnitude = 30;
	let startsFiltered = [];
	$: starsFiltered = starsRaw.filter(star => star.mag <= maxMagnitude);

	//Constelation Filter
	let selectedCons = [];
	let linesFiltered = {};
	let consPos = {X: 0, Y: 0};
	$: linesFiltered = linesRaw.filter(
		([A, B]) => selectedCons.includes(A.con) && selectedCons.includes(B.con)
	)
	$: customAngle = consPos;

	//Place finder
	let userCoordinates = {lat: 0, lon: 0};
	$: customAngle = {X: -(userCoordinates.lat * Math.PI)/180, Y: 0}
</script>

<div class="all">
	<div class="filters">
		<h2>Aqui entram os filtros</h2>
		<ConstelationFilter starsRaw = {starsRaw} bind:selectedCons = {selectedCons} bind:consPosition = {consPos}/>
		<p>posição saída da constelation filter: {consPos.X}, {consPos.Y}</p>
		<MagnitudeFilter starsRaw = {starsRaw} bind:maxMagnitude = {maxMagnitude}/>
		<p>Magnitude máxima de saída da magnitude filter: {maxMagnitude}</p>
		<LocationFinder bind:coordinates = {userCoordinates}/>
		<p>posição saída da location finder: {userCoordinates.lat}, {userCoordinates.lon}</p>
	</div>

	<div class="viz">
		<h2>Aqui entra a viz</h2>
		<StarsPlot starsRaw = {starsFiltered}
				   linesRaw = {linesFiltered}
				   customAngle = {customAngle}
				   size = {800}
				   />
	</div>
</div>

<style>
	.all {
		display: grid;
		grid-template-columns: 25% 70%;
		padding-left: 1%;
	}
</style>
