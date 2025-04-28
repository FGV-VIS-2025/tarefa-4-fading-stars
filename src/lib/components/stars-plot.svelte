<script>
	import consMap from "$lib/data/constellations-map.json";
	import * as d3 from "d3";
	import { computePosition, autoPlacement, offset } from "@floating-ui/dom";
	import { onMount } from "svelte";

	export let starsRaw = [];
	export let linesRaw = [];
	export let customAngle;
	export let size = 800;

	//SVG canvas spec
	let width, height;
	$: width = size;
	$: height = size;
	let margin = { v: 20, h: 20 };
	//Localization
	let angle = { X: 0, Y: 0 };
	let dragSpeed = 0.002;

	//D3 scale creation
	const graticule = d3.geoGraticule10();
	const sphere = { type: "Sphere" };

	$: xScale = d3.scaleLinear([-1, 1], [margin.v, width - margin.v]);
	$: yScale = d3.scaleLinear([-1, 1], [margin.h, height - margin.h]);

	//Mouse movement handlers
	onMount(() => {
		const dragFun = d3.drag().on("drag", (event) => {
			angle.X -= event.dy * dragSpeed;
			angle.Y -= event.dx * dragSpeed;
		});

		d3.select("svg").call(dragFun);
	});

	function rotate(d, angle) {
		let x, y, z;
		x = d.x_proj * Math.cos(angle.Y) - d.z_proj * Math.sin(angle.Y);
		z = d.x_proj * Math.sin(angle.Y) + d.z_proj * Math.cos(angle.Y);
		y = d.y_proj * Math.cos(angle.X) - z * Math.sin(angle.X);
		z = d.y_proj * Math.sin(angle.X) + z * Math.cos(angle.X);

		return { ...d, x: x, y: y, z: z };
	}

	//Star filter by movement and then projection
	$: stars = starsRaw
		.map((d) => rotate(d, angle))
		.filter((star) => star.z >= 0 && star.proper != "Sol");
	$: lines = linesRaw
		.map(([a, b]) => [rotate(a, angle), rotate(b, angle)])
		.filter(([a, b]) => a.z >= 0 && b.z >= 0);
	$: console.log(angle);
	$: projection = d3
		.geoOrthographic()
		.scale(width / 2 - margin.v, height / 2 - margin.h)
		.translate([width / 2, height / 2])
		.rotate([angle.Y * (-180 / Math.PI), angle.X * (180 / Math.PI)]);
	$: pathGenerator = d3.geoPath(projection);

	//Tooltip handlers
	let hoveredIndex = -1;
	$: hoveredStar = stars[hoveredIndex] ?? hoveredStar ?? {};
	let cursorPos = { x: 0, y: 0 },
		tooltipPos = { x: 0, y: 0 };
	let starTooltip;

	let transitionTimer;
	$: if (customAngle.X != 0 || customAngle.Y != 0) {
		const interpolator = d3.interpolateObject(angle, customAngle);

		const duration = 500;
		const start = Date.now();

		transitionTimer = d3.timer(() => {
			const elapsed = Date.now() - start;
			const t = Math.min(1, elapsed / duration);

			angle = interpolator(t);

			if (t >= 1) {
				transitionTimer.stop();
			}
		});
		customAngle = {X: 0, Y: 0};
	}

	$: {
		angle.X = Math.max(-Math.PI / 2, Math.min(Math.PI / 2, angle.X));
		angle.Y = ((angle.Y % (2 * Math.PI)) + 2 * Math.PI) % (2 * Math.PI);
	}

	async function mouseTooltipHandler(index, evt) {
		let hoveredDot = evt.target;
		if (evt.type == "mouseenter") {
			hoveredIndex = index;
			cursorPos = { x: evt.x, y: evt.y };
			tooltipPos = await computePosition(hoveredDot, starTooltip, {
				strategy: "fixed",
				middleware: [offset(5), autoPlacement()],
			});
		} else if (evt.type == "mouseleave") {
			hoveredIndex = -1;
		}
	}
</script>

<!-- svg used as canvas for d3 plotting -->
<svg {width} {height} viewBox="0 0 {width} {height}">
	<path
		d={pathGenerator(graticule)}
		fill="none"
		stroke="#444"
		stroke-width="1.5"
	/>
	<g class="constellation-lines">
		{#each lines as [starA, starB]}
			<line
				x1={xScale(starA.x)}
				y1={yScale(starA.y)}
				x2={xScale(starB.x)}
				y2={yScale(starB.y)}
				stroke="#aaa"
				stroke-width="1"
			/>
		{/each}
	</g>
	<g class="stars">
		{#each stars as star, index}
			<circle
				role="tooltip"
				on:mouseenter={(evt) => mouseTooltipHandler(index, evt)}
				on:mouseleave={(evt) => mouseTooltipHandler(index, evt)}
				cx={xScale(star.x)}
				cy={yScale(star.y)}
				r={((5 * (star.mag - 7)) / (-1.45 - 7)) ** 0.9}
				fill="white"
			/>
		{/each}
	</g>
	<path
		d={pathGenerator(sphere)}
		fill="none"
		stroke="#FFF"
		stroke-width="1.5"
	/>
</svg>

<!-- Tooltip container - use dl tag since its key value-->
<dl
	class="info tooltip"
	hidden={hoveredIndex === -1}
	style="top: {tooltipPos.y}px; left: {tooltipPos.x}px"
	bind:this={starTooltip}
>
	{#if hoveredStar.proper != null}
		<dt>Nome</dt>
		<dd>{hoveredStar.proper}</dd>
		<dd></dd>{/if}

	<dt>Magnitude aparente</dt>
	<dd>{hoveredStar.mag}</dd>

	<dt>Magnitude absoluta</dt>
	<dd>{hoveredStar.absmag}</dd>

	<!--	<dt>Temperatura</dt>
	<dd>{hoveredStar.temp} K</dd>-->

	{#if hoveredStar.dist != null}
		<dt>Luminosidade</dt>
		<dd>{hoveredStar.lum.toFixed(0)}x a do Sol</dd>
	{/if}

	{#if hoveredStar.dist < 100000}
		<dt>Distância da Terra</dt>
		<dd>{(hoveredStar.dist * 3.262).toFixed(2)} anos-luz</dd>
	{/if}

	{#if hoveredStar.con != null}
		<dt>Constelação</dt>
		<dd>{consMap[hoveredStar.con]}</dd>
	{/if}
</dl>

<style>
	.info {
		margin: 0;

		display: grid;
		grid-template-columns: 2;

		padding: 5px;

		background-color: #393939e0;
		box-shadow: 1px 1px 2px 2px #60606050;
		border-radius: 4px;

		font-size: 80%;

		transition-duration: 300ms;
		transition-property: opacity, visibility;
		&[hidden]:not(:hover, :focus-within) {
			opacity: 0;
			visibility: hidden;
		}
	}

	.info dt {
		grid-column: 1;
		grid-row: auto;
		text-transform: uppercase;
		font-weight: bold;
	}

	.info dd {
		grid-column: 2;
		grid-row: auto;
		font-weight: 400;
	}

	.tooltip {
		position: fixed;
		top: 1em;
		left: 1em;
	}
</style>
