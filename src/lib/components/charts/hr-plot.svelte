<script>
	import * as d3 from "d3";

	import starsRaw from "$lib/data/hr-data.json";
	export let size = 800;
	export let action;

	//SVG canvas spec
	let width, height;
	$: width = size;
	$: height = size;
	let margin = { v: 40, h: 40 };

	const stars = starsRaw.filter(
		(d) => d.tem != null && d.lum < 100000 && d.ci <= 2.19,
	);

	const tempExtent = [
		Math.max(...stars.map((stars3) => stars3.tem)),
		Math.min(...stars.map((stars3) => stars3.tem)),
	];
	const lumExtent = [
		Math.max(...stars.map((stars3) => stars3.lum)),
		Math.min(...stars.map((stars3) => stars3.lum)),
	];
	const cinExtent = [
		-0.39, // Math.min(...stars.map((stars3) => stars3.ci)),
		2.19, // Math.max(...stars.map((stars3) => stars3.ci)),
	];
	const magExtent = [
		-7, // Math.min(...stars.map((stars3) => stars3.absmag)),
		17, // Math.max(...stars.map((stars3) => stars3.absmag)),
	];

	$: xScaleLin = d3.scaleLinear(cinExtent, [margin.h, height - margin.h]);
	$: yScaleLin = d3.scaleLinear(magExtent, [margin.h, height - margin.h]);

	$: xScaleLog = d3.scaleLinear().range([margin.v, width - margin.v]);
	$: yScaleLog = d3.scaleLog(
		yScaleLin.domain().map((m) => Math.pow(10, 4.83 - m)),
		yScaleLin.range(),
	);

	function color(temperature) {
		const u = 8464 / temperature;
		return (u - 2.1344 + Math.hypot(0.9936, u)) / 1.6928;
	}

	let xAxisLog, yAxisLog, xAxisLin, yAxisLin;
	const temperatures = [3000, 4000, 5000, 6000, 7000, 8000, 9000, 20000];
	$: {
		d3.select(xAxisLog).call(
			d3
				.axisTop(xScaleLin)
				.tickValues(temperatures.map(color))
				.tickFormat((_, i) => temperatures[i].toLocaleString("pt")),
		);

		d3.select(yAxisLog).call(d3.axisLeft(yScaleLog));

		d3.select(xAxisLin).call(d3.axisBottom(xScaleLin).ticks(null, "+f"));
		d3.select(yAxisLin).call(d3.axisRight(yScaleLin).ticks(null, "+"));
	}

	let expr = null;
	$: {
		switch (action) {
			case "mainseq":
				expr = "mainseq";
				break;
			case "dwarf":
				expr = "dwarf";
				break;
			case "subgig":
				expr = "subgig";
				break;
			case "gig":
				expr = "gig";
				break;
			case "supgig":
				expr = "supgig";
				break;
			default:
				expr = null;
		}
	}

	let showCross = false;
	let x, y;

	function updateBackground(textId, bgId) {
		const text = d3.select(textId);
		const bbox = text.node().getBBox();
		d3.select(bgId)
			.attr("x", bbox.x - 4)
			.attr("y", bbox.y - 2)
			.attr("width", bbox.width + 8)
			.attr("height", bbox.height + 4);
	}

	function mouseMove(event) {
		showCross = true;
		[x, y] = d3.pointer(event);

		const ci = xScaleLin.invert(x);
		const tem = 4600 * (1 / (0.92 * ci + 1.7) + 1 / (0.92 * ci + 0.62));
		const absmag = yScaleLin.invert(y);
		const lum = yScaleLog.invert(y);

		d3.select("#ci-value")
			.text(`${ci.toFixed(2)}`)
			.attr("x", x)
			.attr("y", height - margin.h - offset)
			.attr("text-anchor", "middle");

		d3.select("#lum-value")
			.text(`${lum.toExponential(2)}L⊙`)
			.attr("x", margin.v + offset)
			.attr("y", y)
			.attr("text-anchor", "start");

		d3.select("#absmag-value")
			.text(`${absmag.toFixed(2)}M`)
			.attr("x", width - margin.v - offset)
			.attr("y", y)
			.attr("text-anchor", "end");

		d3.select("#tem-value")
			.text(`${tem.toFixed(0)}K`)
			.attr("x", x)
			.attr("y", margin.h + offset)
			.attr("text-anchor", "middle");

		updateBackground("#ci-value", "#ci-bg");
		updateBackground("#lum-value", "#lum-bg");
		updateBackground("#absmag-value", "#absmag-bg");
		updateBackground("#tem-value", "#tem-bg");
	}

	function mouseLeave(event) {
		showCross = false;
	}

	let offset = 55;
</script>

<!-- svg used as canvas for d3 plotting -->
<svg
	{width}
	{height}
	viewBox="0 0 {width} {height}"
	id="hr-plot"
	class="svg-plot"
>
	<g class="stars">
		{#each stars as star}
			<circle
				cx={xScaleLin(star.ci)}
				cy={yScaleLin(star.absmag)}
				r={0.8}
				fill={star.rgb}
				class="star"
				class:visible={expr == null || expr == star.class}
			/>
		{/each}
	</g>

	{#if showCross}
		<g>
			<line
				id="v-line"
				class="crosshair"
				x1={x}
				x2={x}
				y1={height - margin.h}
				y2={margin.h}
			/>
			<line
				id="h-line"
				class="crosshair"
				y1={y}
				y2={y}
				x1={margin.v}
				x2={width - margin.v}
			/>

			<g id="ci-label">
				<rect id="ci-bg" fill="var(--accent-black)"></rect>
				<text id="ci-value" dy="0.5ch" fill="white"></text>
			</g>
			<g id="lum-label">
				<rect id="lum-bg" fill="var(--accent-black)"></rect>
				<text id="lum-value" dy="0.5ch" fill="white"></text>
			</g>
			<g id="absmag-label">
				<rect id="absmag-bg" fill="var(--accent-black)"></rect>
				<text id="absmag-value" dy="0.5ch" fill="white"></text>
			</g>
			<g id="tem-label">
				<rect id="tem-bg" fill="var(--accent-black)"></rect>
				<text id="tem-value" dy="0.5ch" fill="white"></text>
			</g>
		</g>
	{/if}

	<g transform="translate(0, {margin.h})" bind:this={xAxisLog}></g>
	<g transform="translate({margin.v}, 0)" bind:this={yAxisLog}></g>

	<g transform="translate(0, {height - margin.h})" bind:this={xAxisLin}></g>
	<g transform="translate({width - margin.v}, 0)" bind:this={yAxisLin}></g>

	<text
		transform={`translate(${width - offset},${height / 2}) rotate(-90)`}
		dy="0.5ch"
	>
		<tspan>&#9664; escuro</tspan>
		Magnitude Absoluta (M)
		<tspan>claro &#9654;</tspan>
	</text>
	<text transform={`translate(${width / 2},${offset})`} dy="0.5ch">
		<tspan>&#9664; quente</tspan>
		Temperatura (K)
		<tspan>fria &#9654;</tspan>
	</text>
	<text transform={`translate(${width / 2},${height - offset})`} dy="0.5ch">
		<tspan>&#9664; azul</tspan>
		Cor (B-V)
		<tspan>vermelha &#9654;</tspan>
	</text>
	<text
		transform={`translate(${offset},${height / 2}) rotate(-90)`}
		dy="0.5ch"
	>
		<tspan>&#9664; escuro</tspan>
		log Luminosidade (L⊙)
		<tspan>claro &#9654;</tspan>
	</text>

	<rect
		role="complementary"
		x={margin.v}
		y={margin.h}
		width={width - 2 * margin.v}
		height={height - 2 * margin.h}
		opacity="0"
		style="cursor: none;"
		on:mousemove={mouseMove}
		on:mouseleave={mouseLeave}
	/>
</svg>

<style>
	svg text {
		fill: white;
		font-family: "Courier New", Courier, monospace;
		font-weight: bold;
		font-size: 10pt;
	}

	svg text tspan {
		fill: gray;
		text-anchor: middle;
		font-family: "Courier New", Courier, monospace;
		/* font-weight: bold; */
		font-size: 9pt;
	}

	.crosshair {
		stroke: crimson;
		stroke-width: 2;
		stroke-dasharray: 5 2;
		pointer-events: none;
	}

	.star {
		opacity: 0;
	}

	.star.visible {
		opacity: 1;
	}
</style>
