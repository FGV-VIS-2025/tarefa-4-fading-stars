<script>
	import * as d3 from "d3";

	import starsRaw from "$lib/data/hr-data.json";
	export let size = 800;
	export let stars;
	export let action;

	//SVG canvas spec
	let width, height;
	$: width = size;
	$: height = size;
	let margin = { v: 40, h: 40 };

	const starsCopy = starsRaw
		.filter((d) => d.tem != null && d.lum <= 100000 && d.ci <= 2.19)

	const tempExtent = [
		Math.max(...starsCopy.map((stars3) => stars3.tem)),
		Math.min(...starsCopy.map((stars3) => stars3.tem)),
	];
	const lumExtent = [
		Math.max(...starsCopy.map((stars3) => stars3.lum)),
		Math.min(...starsCopy.map((stars3) => stars3.lum)),
	];

	const cinExtent = [
		Math.min(...starsCopy.map((stars3) => stars3.ci)),
		Math.max(...starsCopy.map((stars3) => stars3.ci)),
	];

	const magExtent = [
		Math.min(...starsCopy.map((stars3) => stars3.absmag)),
		Math.max(...starsCopy.map((stars3) => stars3.absmag)),
	];

	$: xScaleLog = d3.scaleLog(tempExtent, [margin.v, width - margin.v]);
	$: yScaleLog = d3.scaleLog(lumExtent, [margin.h, height - margin.h]);
	
	$: xScaleLin = d3.scaleLinear(cinExtent, [margin.h, height - margin.h]);
	$: yScaleLin = d3.scaleLinear(magExtent, [margin.h, height - margin.h]);
	
	$: stars = starsCopy;

	let xAxisLog, yAxisLog, xAxisLin, yAxisLin;
	$: {
		d3.select(xAxisLog).call(d3.axisTop(xScaleLog));
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
</script>

<!-- svg used as canvas for d3 plotting -->
<svg {width} {height} viewBox="0 0 {width} {height}" id="hr-plot">
	<g transform="translate(0, {margin.h})" bind:this={xAxisLog}></g>
    <g transform="translate({margin.v}, 0)" bind:this={yAxisLog}></g>
	
	
    <g transform="translate(0, {height-margin.h})" bind:this={xAxisLin}></g>
    <g transform="translate({width-margin.v}, 0)" bind:this={yAxisLin}></g>

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
</svg>



<style>
	.star {
		opacity: 0;
	}

	.star.visible {
		opacity: 1;
	}
</style>
