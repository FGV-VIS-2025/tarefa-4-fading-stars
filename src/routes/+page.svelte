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
	let snapCurr = null;
	let plot = "hr";

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
	let consPos = { position: { X: 0, Y: 0 }, name: "" };
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
</script>

<svelte:window bind:innerWidth bind:innerHeight />

<div class="debugbox"></div>

<div class="scroll">
	<div class="scroll__text">
		<div class="step">
			<h1><u>Diagrama HR</u></h1>
			<p>
				O <i>diagrama de Hertzprung-Russell (HR)</i> é muito conhecido na área da astronomia. 
				Ele mostra a relação entre a magnitude absoluta ou luminosidade pela cor de uma estrela ou
				temperatura efetiva. A criação desse diagrama foi um grande
				avanço para o entendimento do ciclo de vida estelar.
			</p>
			<p>
				O ciclo se inicia a partir da formação de uma <i>proto-estrela</i>, e
				quando a fusão nuclear se inicia a estrela entra na
				<span class="highlight" data-action="mainseq">sequência principal.</span>
				Todas estrelas entram nesse grupo em algum momento da sua vida.
			</p>
			<p>
				O <span class="highlight" data-action="sun">Sol</span> por exemplo,
				é uma estrela pequena da sequência principal.
			</p>
			<p>
				Algumas estrelas são classificadas como 
				<span class="highlight" data-action="gig">gigantes</span>
				podendo ser ou <i>gigantes azuis</i> ou
				<i>gigantes vermelhas</i>. No caso das <i>vermelhas</i>, elas
				surgem quando uma estrela pequena da sequência principal esgota
				seu hidrogênio. Se a estrela for massiva, ao esgotar seu
				hidrogênio ela se torna uma
				<span class="highlight" data-action="supgig">super-gigante</span>
				<i>vermelha</i>.
			</p>
			<p>
				Por fim, surgem em sequência as
				<span class="highlight" data-action="dwarf">anãs-brancas</span>
				que são o estágio final de uma estrela que foi
				<i>gigante vermelha</i>
			</p>
			<p>
				Algo curioso, é que estrelas mais quentes possuem tonalidade 
				<span style="color: #b5c4f0">azul</span>, 
				enquanto estrelas mais frias tem tonalidade
				<span style="color: #d97923">vermelha</span>. Note também que estrelas mais brilhantes tem magnitude menor.
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
				{userCoordinates}
			/>
			<!-- 			<p>posição saída da constelation filter: {consPos.X}, {consPos.Y}</p> -->
			<LocationFinder
				bind:coordinates={userCoordinates}
				bind:action={placeAction}
			/>
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
				<div class="plot" class:visible={plot == "hr"}>
					<HRPlot size={0.9 * innerHeight} action={highlightAction} />
				</div>
				<div class="plot" class:visible={plot == "sphere"}>
					<StarsPlot
						starsRaw={starsFiltered}
						linesRaw={linesFiltered}
						{customAngle}
						size={0.9 * innerHeight}
						action={highlightAction}
						bind:stars={visibleStars}
						constellation={consPos.name}
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
