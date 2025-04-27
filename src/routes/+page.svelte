<script>
	//Components
	import StarsPlot from "$lib/components/stars-plot.svelte";
	import LocationFinder from "$lib/components/get-location.svelte";
	import MagnitudeFilter from "$lib/components/filter-magnitude.svelte";
	//Data
	import starsRaw from "$lib/data/stars-data.json";

	//Magnitude Filter
	let maxMagnitude = 30;
	let startsFiltered = [];
	$: starsFiltered = starsRaw.filter(star => star.mag <= maxMagnitude);

	//Place finder
	let userCoordinates = {lat: 0, lon: 0};

</script>

<div class="all">
	<div class="filters">
		<h2>Aqui entram os filtros</h2>
		<LocationFinder bind:coordinates = {userCoordinates}/>
		<p>Você está em {userCoordinates.lat}, {userCoordinates.lon}</p>
		<MagnitudeFilter starsRaw = {starsRaw} bind:maxMagnitude = {maxMagnitude}/>
	</div>

	<div class="viz">
		<h2>Aqui entra a viz</h2>
		<StarsPlot starsRaw = {starsFiltered}/>
	</div>
</div>

<style>
	.all {
		display: grid;
		grid-template-columns: 30% 70%;
	}
</style>
