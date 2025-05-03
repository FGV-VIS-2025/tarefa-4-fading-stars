<script>
	//Components - visualizations
	import scrollama from "scrollama";

	import StarsPlot from "$lib/components/charts/stars-plot.svelte";
	import HRPlot from "$lib/components/charts/hr-plot.svelte";
	
	import StarsHistogram from "$lib/components/star-histogram.svelte";
	//Components - filters
	import LocationFinder from "$lib/components/get-location.svelte";
	import MagnitudeFilter from "$lib/components/filter-magnitude.svelte";
	import ConstelationFilter from "$lib/components/filter-constelations.svelte";
	//Svelte on mount
	import { onMount } from "svelte";
	//D3
	import * as d3 from "d3";
	//Data
	import starsRaw from "$lib/data/stars-data.json";
	import linesRaw from "$lib/data/lines-data.json";

	let innerWidth;
	let innerHeight;

	let highlightAction = null;
	let vizElement = null;
	onMount(() => {
		innerWidth = window.innerWidth;
		innerHeight = window.innerHeight;

		const sizeSteps = document.querySelectorAll(".step").length;
		const highlights = document.querySelectorAll(".highlight");
		vizElement = document.getElementById("viz");

		highlights.forEach((span) => {
			const action = span.dataset.action;
			span.addEventListener("mouseover", () => {
				highlightAction = action;
			});
			span.addEventListener("mouseout", () => {
				highlightAction = null;
			});
		});

		const scrollInstance = new scrollama();

		scrollInstance
			.setup({
				step: ".step",
				offset: 0.5,
				// debug: true,
			})
			.onStepEnter(({ element, direction, index }) => {
				element.classList.add("focused");
				snapCurr = index;
				if (direction === "down") {
					vizElement.classList.add("focused");
				}
				if (index === sizeSteps - 1 && direction === "up") {
					vizElement.classList.add("focused");
				}

				if (index <= 0) {
					plot = "hr";
				} else if (index >= 0) {
					plot = "sphere";
				}
			})
			.onStepExit(({ element, direction, index }) => {
				element.classList.remove("focused");
				if (direction === "up" && index == 0) {
					vizElement.classList.remove("focused");
				}
				if (index === sizeSteps - 1 && direction === "down") {
					vizElement.classList.remove("focused");
				}
			});
	});
	$: console.log("highlightaction: ", highlightAction);
	$: console.log(innerHeight, innerWidth);

	let customAngle = { X: 0, Y: 0 };
	let fixedYabsmag = 2300;
	let fixedYtemp = 1450;
	//Magnitude Filter (also affects histogram vars)
	let maxMagnitude = 30;
	let starsFiltered = [];
	let starsFilteredIds = [];
	let percentageFiltered = 1;
	let starsRawTotalCount = starsRaw.length;
	$: {
		starsFiltered = starsRaw.filter((star) => star.mag <= maxMagnitude);
		starsFilteredIds = starsFiltered.map((star) => star.id);
		percentageFiltered = starsFiltered.length / starsRawTotalCount;
		fixedYabsmag = 2130 * percentageFiltered;
		fixedYtemp = 1360 * percentageFiltered;
	}

	//Constelation Filter
	let selectedCons = [];
	let linesFiltered = {};
	let consPos = {position: { X: 0, Y: 0 }, name: ""};
	$: linesFiltered = linesRaw.filter(
		([A, B]) =>
			selectedCons.includes(A.con) &&
			selectedCons.includes(B.con) &&
			starsFilteredIds.includes(A.id) &&
			starsFilteredIds.includes(B.id),
	);
	$: {
		customAngle = consPos.position;
		console.log(consPos);
	}

	//Place finder
	let userCoordinates = { lat: 0, lon: 0 };
	let placeAction = null;
	$: customAngle = { X: -(userCoordinates.lat * Math.PI) / 180, Y: 0 };
	$: highlightAction = placeAction;

	//histogram inputs
	let visibleStars = starsRaw;

	let snapCurr = null;

	let plot = "hr";
</script>

<svelte:window bind:innerWidth bind:innerHeight />

<div class="debugbox"></div>

<div class="scroll">
	<div class="scroll__text">
		<div class="step">
			<p>
				O diagrama de Hertzprung-Russell, também conhecido como diagrama
				HR, é um tipo de visualização muito conhecida na astronomia. Ele
				consiste em um scatterplot que mostra relações entre magnitude
				(luminosidade) absoluta de diversas estrelas com suas
				respectivas temperaturas.
			</p>
			<p>
				Essa forma de representação ignora a posição das estrelas em
				relação à Terra, já que seu objetivo é mostrar que existem
				relações entre os atributos plotados das estrelas. Essas
				relações evidenciam o ciclo de vida de uma estrela.
			</p>
			<p>
				A formação de uma estrela inicia com a concentração de gás
				hidrogênio em um único ponto. A temperatura e luminosidade desse
				ponto, que é a estrela em formação, são altíssimos. Essas são as
				estrelas localizadas no canto superior esquerdo do diagrama
			</p>
			<p>
				Depois de milhões de anos nesse estado, o gás hidrogênio vai se
				esgotando, e a estrela pode seguir por dois caminhos diferentes:
				se tornar uma anã branca ou uma gigante vermelha.

				<span class="highlight" data-action="mainseq">highlight</span>
				<span class="highlight" data-action="supgig">highlight</span>
				<span class="highlight" data-action="gig">highlight</span>
				<span class="highlight" data-action="dwarf">highlight</span>
			</p>
		</div>
		<div class="step">
			Hello, I'm testing a highlight: <span
				class="highlight"
				data-action="latitude">highlight</span
			>

			Hello, I'm testing another highlight:
			<span class="highlight" data-action="rotate">highlight</span>
		</div>
		<div class="step" style="padding-top:10%;">
			<ConstelationFilter
				{starsRaw}
				bind:selectedCons
				bind:consPosition={consPos}
			/>
			<MagnitudeFilter
				{starsRaw}
				bind:maxMagnitude
				percentage={percentageFiltered}
			/>
			<!-- 			<p>posição saída da constelation filter: {consPos.X}, {consPos.Y}</p> -->
			<LocationFinder bind:coordinates={userCoordinates} bind:action = {placeAction}/>
			<!-- 			<p>Magnitude máxima de saída da magnitude filter: {maxMagnitude}</p> -->
			<!-- 			<p>posição saída da location finder: {userCoordinates.lat}, {userCoordinates.lon}</p> -->
		</div>
		<div class="step" style="padding-top:5%;">
			{#if innerHeight != null}
				<h3>Distribuição das temperaturas das estrelas visíveis</h3>
				<StarsHistogram
					starsRaw={visibleStars}
					variable="tem"
					label="Temperatura (K)"
					fixedY={fixedYtemp}
					dims={{
						height: 0.35 * innerHeight,
						width: innerWidth * 0.3,
					}}
				/>
				<h3>
					Distribuição das magnitudes absolutas das estrelas visíveis
				</h3>
				<StarsHistogram
					starsRaw={visibleStars}
					variable="absmag"
					label="Magnitude absoluta"
					fixedY={fixedYabsmag}
					dims={{
						height: 0.35 * innerHeight,
						width: innerWidth * 0.3,
					}}
				/>
			{/if}
		</div>
	</div>
	<div class="scroll__graphic">
		<div id="viz">
			{#if innerHeight != null}
				<div class="plot" class:visible={plot=="hr"}>
					<HRPlot
					size={0.9 * innerHeight}
					action={highlightAction}
					/>
				</div>
				<div class="plot" class:visible={plot=="sphere"}>
					<StarsPlot
					starsRaw={starsFiltered}
					linesRaw={linesFiltered}
					{customAngle}
					size={0.9 * innerHeight}
					action={highlightAction}
					bind:stars={visibleStars}
					constellation={consPos.name}

					class="plot"
					/>
				</div>
			{/if}
		</div>
	</div>
</div>

<div class="debugbox"></div>

<style>
	.debugbox {
		scroll-snap-align: start;
		background-color: antiquewhite;
		height: 60vh;
	}

	.highlight {
		background-color: #f1f1f1;
		color: #000;
		display: inline-block;
		margin: 3px 0;
		padding: 3px;
		font-weight: bold;
		cursor: default;
	}

	.scroll {
		padding: 0 5%;
		position: relative;
		display: flex;
	}

	.scroll > * {
		-webkit-box-flex: 1;
		-ms-flex: 1;
		flex: 1;
	}

	.plot {
		display: none;
		opacity: 0;
	}

	.plot.visible {
		display: block;
		opacity: 1;
	}

	.step:not(.focused) {
		pointer-events: none;
		filter: blur(2px);
		opacity: 0.5;
	}

	#viz:not(.focused) {
		filter: blur(2px);
		opacity: 0.5;
	}

	#viz {
		transition:
			filter 0.3s ease,
			opacity 0.3s ease;
	}

	.scroll__text {
		margin: 10% 0;
		padding-right: 100px;
		max-width: 40%;
	}

	.scroll__graphic {
		position: sticky;
		top: 0;
		height: 100vh;

		display: flex;
		align-items: center;
		justify-content: center;
	}

	.step {
		padding: 100px 0;
		scroll-snap-align: center;
		scroll-snap-stop: always;
		transition:
			filter 0.3s ease,
			opacity 0.3s ease;
	}

</style>
