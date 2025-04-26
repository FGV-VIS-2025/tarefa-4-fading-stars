<script>
	import StarsPlot from "$lib/components/stars-plot.svelte";
	import LocationFinder from "$lib/components/get-location.svelte";
	let userCoordinates = {lat: 0, lon: 0};
</script>

<div class="all">
	<div class="filters">
		<h2>Aqui entram os filtros</h2>
		<LocationFinder bind:coordinates = {userCoordinates}/>
		<p>Você está em {userCoordinates.lat}, {userCoordinates.lon}</p>
	</div>

	<div class="viz">
		<h2>Aqui entra a viz</h2>
		<StarsPlot local = bind:inputedLocation/>
	</div>
</div>

<style>
	.all {
		display: grid;
		grid-template-columns: 40% 45%;
	}
</style>
