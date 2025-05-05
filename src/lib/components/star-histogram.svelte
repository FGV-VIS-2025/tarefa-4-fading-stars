<script>
	import * as d3 from "d3";
	import { computePosition, autoPlacement, offset } from "@floating-ui/dom";
	import { onMount } from "svelte";

	export let starsRaw = []; //input with stars to be displayed
	export let dims = {width: 400, height: 300}; //input of size of viz
	export let variable; //input of which variable to use for hist plotting
	export let label; //label to show in x axis
	export let fixedY = 0;

	//SVG canvas spec
	let width, height;
	$: width = dims.width;
	$: height = dims.height;
	let margin = {left: 40, right: 20, top: 20, bottom: 30};

	//Bin data distribution
	let bins;
	$: {
		bins = d3.bin().thresholds(30)
			.value(star => star[variable])
			(starsRaw.filter(star => star[variable] != null));
	}

	//D3 scale creation
	let xScale, yScale;
	$: {
		xScale = d3.scaleLinear()
			.domain([bins[0].x0, bins[bins.length - 1].x1])
			.range([margin.left, width - margin.right]);
		yScale = d3.scaleLinear()
			.domain([0, (d3.max(bins, b => b.length) > fixedY)? d3.max(bins, b => b.length) : fixedY])
			.range([height - margin.bottom, margin.top]);
		};

	let xAxis, yAxis;
	$: {
		d3.select(xAxis).call(d3.axisBottom(xScale));
		d3.select(yAxis).call(d3.axisLeft(yScale));
	}

</script>

<!-- svg used as canvas for d3 plotting -->
<svg {width} {height} viewBox="0 0 {width} {height}" id="hist">
	<g transform = "translate(0, {height - margin.bottom})" bind:this={xAxis}>
		<text x={width/2 + margin.left}
			  y={margin.bottom - 3}
			  fill="currentcolor"
			  text-anchor="end">{label}</text>
	</g>
    <g transform = "translate({margin.left}, 0)" bind:this={yAxis}>
		<text x={-margin.left}
			  y="10"
			  fill="currentcolor"
			  text-anchor="start">
			  Quantidade de estrelas
			  </text>
    </g>
	<g id="rects" fill="white">
	{#each bins as d}
		<rect x="{xScale(d.x0) + 1}"
			  width="{xScale(d.x1) - xScale(d.x0)}"
			  y="{yScale(d.length)}"
			  height="{yScale(0) - yScale(d.length)}"
		/>
	{/each}
	</g>
</svg>

<style>

</style>
