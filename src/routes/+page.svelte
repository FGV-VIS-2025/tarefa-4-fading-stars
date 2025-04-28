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

	let starsRawTotalCount = starsRaw.length;
	//Magnitude Filter
	let maxMagnitude = 30;
	let starsFiltered = [];
	let starsFilteredIds = [];
	let percentageFiltered = 1;
	$: starsFiltered = starsRaw.filter(star => star.mag <= maxMagnitude);
	$: starsFilteredIds = starsFiltered.map(star => star.id);
	$: percentageFiltered = starsFiltered.length/starsRawTotalCount;

	//Constelation Filter
	let selectedCons = [];
	let linesFiltered = {};
	let consPos = {X: 0, Y: 0};
	$: linesFiltered = linesRaw.filter(
		([A, B]) => selectedCons.includes(A.con) &&
					selectedCons.includes(B.con) &&
					starsFilteredIds.includes(A.id) &&
					starsFilteredIds.includes(B.id)
	)
	$: customAngle = consPos;

	//Place finder
	let userCoordinates = {lat: 0, lon: 0};
	$: customAngle = {X: -(userCoordinates.lat * Math.PI)/180, Y: 0}
</script>

<div class="container">
	<div class="left">
		<div class="snap-item">
			<ConstelationFilter starsRaw = {starsRaw} bind:selectedCons = {selectedCons} bind:consPosition = {consPos}/>
<!-- 			<p>posição saída da constelation filter: {consPos.X}, {consPos.Y}</p> -->
			<MagnitudeFilter starsRaw = {starsRaw} bind:maxMagnitude = {maxMagnitude} percentage = {percentageFiltered}/>
<!-- 			<p>Magnitude máxima de saída da magnitude filter: {maxMagnitude}</p> -->
			<LocationFinder bind:coordinates = {userCoordinates}/>
<!-- 			<p>posição saída da location finder: {userCoordinates.lat}, {userCoordinates.lon}</p> -->
		</div>
		<div class="snap-item">
			bla bla bla
		</div>
		<div class="snap-item">
			blu blu blu
		</div>
	</div>
	<div class="right">
		<div id="viz">
		<StarsPlot starsRaw = {starsFiltered}
				   linesRaw = {linesFiltered}
				   customAngle = {customAngle}
				   size = {800}
				   />
		</div>
	</div>
</div>

<style>
.container {
	height: 100%;
	width: 100%;
    overflow: hidden;
    display: flex;
    overflow: hidden;
}

.left {
	flex: 1;
	height: 100vh;
	max-width: 40%;
	overflow-y: scroll;
	scroll-snap-type: y mandatory;
}

.right {
	width: 60%;
	align-items: center;
	justify-content: center;
	position: fixed;
	display: flex;
	right: 0;
	height: 100vh;
}

.snap-item {
	padding: 100px;
	height: 100vh;
	scroll-snap-align: start;
	border-bottom: 2px solid #ccc;
}

</style>
