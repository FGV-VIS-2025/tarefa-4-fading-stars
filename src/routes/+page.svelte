<script>
	//Components - visualizations
	import scrollama from 'scrollama';

	import StarsPlot from "$lib/components/stars-plot.svelte";
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

	onMount(() => {
		innerWidth = window.innerWidth;
		innerHeight = window.innerHeight;

		const highlights = document.querySelectorAll(".highlight");
		highlights.forEach((span) => {
			const action = span.dataset.action;
			span.addEventListener("mouseover", () => {
				highlightAction = action;
			});
			span.addEventListener("mouseout", () => {
				highlightAction = null;
			});
		});

		const scrollerInstance = new scrollama();

		scrollerInstance
		.setup({
			step: '.snap-item',
			offset: 0.5,
			// debug: true,
		})
		.onStepEnter(({ element, direction, index }) => {
			element.classList.add('focused');
			snapCurr = index;
		})
		.onStepExit(({ element, direction, index }) => {
			element.classList.remove('focused');
		});

		scroller = scrollerInstance;
	});
	$: console.log("highlightaction: ", highlightAction);
	$: console.log(innerHeight, innerWidth);

	let customAngle = { X: 0, Y: 0 };
	let fixedYabsmag = 2300;
	let fixedYtemp = 1450;
	//Magnitude Filter
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
	let consPos = { X: 0, Y: 0 };
	$: linesFiltered = linesRaw.filter(
		([A, B]) =>
			selectedCons.includes(A.con) &&
			selectedCons.includes(B.con) &&
			starsFilteredIds.includes(A.id) &&
			starsFilteredIds.includes(B.id),
	);
	$: customAngle = consPos;

	//Place finder
	let userCoordinates = { lat: 0, lon: 0 };
	$: customAngle = { X: -(userCoordinates.lat * Math.PI) / 180, Y: 0 };

	//histogram inputs
	let visibleStars = starsRaw;

	let snapCurr = 0;
</script>

<svelte:window bind:innerWidth bind:innerHeight />

<div class="container">
	<div class="left">
		<div class="snap-item">
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
			</p>
		</div>
		<div class="snap-item">
			Hello, I'm testing a highlight: <span
				class="highlight"
				data-action="latitude">highlight</span
			>

			Hello, I'm testing another highlight:
			<span class="highlight" data-action="rotate">highlight</span>
		</div>
		<div class="snap-item" style="padding-top:10%;">
			<ConstelationFilter
				{starsRaw}
				bind:selectedCons
				bind:consPosition={consPos}
			/>
			<!-- 			<p>posição saída da constelation filter: {consPos.X}, {consPos.Y}</p> -->
			<MagnitudeFilter
				{starsRaw}
				bind:maxMagnitude
				percentage={percentageFiltered}
			/>
			<!-- 			<p>Magnitude máxima de saída da magnitude filter: {maxMagnitude}</p> -->
			<LocationFinder bind:coordinates={userCoordinates} />
			<!-- 			<p>posição saída da location finder: {userCoordinates.lat}, {userCoordinates.lon}</p> -->
		</div>
		<div class="snap-item" style="padding-top:5%;">
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
	<div class="right">
		<div id="viz">
			{#if innerHeight != null}
				<StarsPlot
					starsRaw={starsFiltered}
					linesRaw={linesFiltered}
					{customAngle}
					size={0.9 * innerHeight}
					{snapCurr}
					action={highlightAction}
					bind:stars={visibleStars}
				/>
			{/if}
		</div>
	</div>
</div>

<style>
	.highlight {
		background-color: #f1f1f1;
		color: #000;
		display: inline-block;
		margin: 3px 0;
		padding: 3px;
		font-weight: bold;
		cursor: pointer;
	}

	.container {
		height: 100%;
		width: 100%;
		overflow: hidden;
		display: flex;
		overflow: hidden;
	}

	.snap-item:not(.focused) {
		filter: blur(2px);
		opacity: 0.5;
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
		scroll-snap-align: center;
		transition: filter 0.3s ease, opacity 0.3s ease;
	}
</style>
