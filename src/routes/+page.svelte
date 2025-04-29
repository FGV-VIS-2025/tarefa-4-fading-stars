<svelte:window bind:innerWidth bind:innerHeight />

<script>
	//Components - visualizations
	import StarsPlot from "$lib/components/stars-plot.svelte";
	import StarsHistogram from "$lib/components/star-histogram.svelte";
	//Components - filters
	import LocationFinder from "$lib/components/get-location.svelte";
	import MagnitudeFilter from "$lib/components/filter-magnitude.svelte";
	import ConstelationFilter from "$lib/components/filter-constelations.svelte"
	//Svelte on mount
	import { onMount } from "svelte";
	//D3
	import * as d3 from "d3";
	//Data
	import starsRaw from "$lib/data/stars-data.json";
	import linesRaw from "$lib/data/lines-data.json";

	let innerWidth;
	let innerHeight;
	onMount(() => {
		innerWidth = window.innerWidth;
		innerHeight = window.innerHeight;
	});
	$: console.log(innerHeight, innerWidth);

	let customAngle = {X: 0, Y: 0};

	//Magnitude Filter
	let maxMagnitude = 30;
	let starsFiltered = [];
	let starsFilteredIds = [];
	let percentageFiltered = 1;
	let starsRawTotalCount = starsRaw.length;
	$: {
		starsFiltered = starsRaw.filter(star => star.mag <= maxMagnitude);
		starsFilteredIds = starsFiltered.map(star => star.id);
		percentageFiltered = starsFiltered.length/starsRawTotalCount;
	};

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

	//histogram inputs
	let visibleStars = starsRaw;

</script>


<div class="container">
	<div class="left">
		<div class="snap-item" style="padding-top:10%;">
			<ConstelationFilter starsRaw = {starsRaw} bind:selectedCons = {selectedCons} bind:consPosition = {consPos}/>
<!-- 			<p>posição saída da constelation filter: {consPos.X}, {consPos.Y}</p> -->
			<MagnitudeFilter starsRaw = {starsRaw} bind:maxMagnitude = {maxMagnitude} percentage = {percentageFiltered}/>
<!-- 			<p>Magnitude máxima de saída da magnitude filter: {maxMagnitude}</p> -->
			<LocationFinder bind:coordinates = {userCoordinates}/>
<!-- 			<p>posição saída da location finder: {userCoordinates.lat}, {userCoordinates.lon}</p> -->
		</div>
		<div class="snap-item" style="padding-top:5%;">
			<h3>Distribuição das temperaturas das estrelas visíveis</h3>
			<StarsHistogram starsRaw = {visibleStars}
							variable = "tem"
							label = "Temperatura (K)"
							dims = {{height: 0.35*innerHeight, width: innerWidth*0.3}}/>
			<h3>Distribuição das magnitudes absolutas das estrelas visíveis</h3>
			<StarsHistogram starsRaw = {visibleStars}
							variable = "absmag"
							label = "Magnitude absoluta"
							dims = {{height: 0.35*innerHeight, width: innerWidth*0.3}}/>
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
				   size = {0.9*innerHeight}
				   bind:stars = {visibleStars}
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
	min-height: 100vh;
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
	min-height: 90vh;
	scroll-snap-align: start;
	border-bottom: 2px solid #ccc;
}

</style>
