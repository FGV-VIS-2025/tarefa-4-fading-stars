<script>
	import consMap from "$lib/data/constellations-map.json";
	import * as d3 from "d3";
	import { computePosition, autoPlacement, offset } from "@floating-ui/dom";
	import { onMount } from "svelte";

	export let starsRaw = [];
	export let linesRaw = [];
	export let customAngle;
	export let size = 800;
	export let stars;
	export let action;
	export let constellation;

	//SVG canvas spec
	$: width = size;
	$: height = size;
	let margin = { v: 40, h: 40 };

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

		d3.select("#celestial-sphere").call(dragFun);
	});

	function rotate(d, angle) {
		const cosY = Math.cos(angle.Y);
		const sinY = Math.sin(angle.Y);
		const cosX = Math.cos(angle.X);
		const sinX = Math.sin(angle.X);

		const x1 = d.x_proj * cosY - d.z_proj * sinY;
		const z1 = d.x_proj * sinY + d.z_proj * cosY;

		const y1 = d.y_proj * cosX - z1 * sinX;
		const z2 = d.y_proj * sinX + z1 * cosX;

		return { ...d, x: x1, y: y1, z: z2 };
	}

	function angleAnimation(newAngle, ignoreY = false) {
		const normalizeAngle = (angle) => {
			return angle % (2 * Math.PI);
		};

		const shortestPath = (current, target) => {
			const normCurrent = normalizeAngle(current);
			const normTarget = normalizeAngle(target);

			let diff1 = normTarget - normCurrent;
			let diff2 = diff1 - 2 * Math.PI;
			if (diff1 < 0) {
				diff2 = diff1 + 2 * Math.PI;
			}

			return Math.abs(diff1) < Math.abs(diff2) ? diff1 : diff2;
		};

		const targetAngle = {
			X: normalizeAngle(angle.X) + shortestPath(angle.X, newAngle.X),
			Y: ignoreY
				? angle.Y
				: normalizeAngle(angle.Y) + shortestPath(angle.Y, newAngle.Y),
		};

		const interpolator = d3.interpolateObject(angle, targetAngle);

		const duration = 500;
		const start = Date.now();

		const transitionTimer = d3.timer(() => {
			const elapsed = Date.now() - start;
			const t = Math.min(1, elapsed / duration);

			if (ignoreY) {
				angle.X = interpolator(t).X;
			} else {
				angle = interpolator(t);
			}

			if (t >= 1) {
				transitionTimer.stop();
			}
		});
	}

	function magAnimation(fromMag, toMag) {
		const interpolator = d3.interpolate(fromMag, toMag);
		const duration = 500;
		const start = Date.now();

		const transitionTimer = d3.timer(() => {
			const elapsed = Date.now() - start;
			const t = Math.min(1, elapsed / duration);

			limitMag = interpolator(t);
			
			if (t >= 1) {
				transitionTimer.stop();
			}
		});
	}

	$: if (customAngle.X != 0 || customAngle.Y != 0) {
		angleAnimation(customAngle);
		customAngle = { X: 0, Y: 0 };
	}

	$: {
		angle.X = Math.max(-Math.PI / 2, Math.min(Math.PI / 2, angle.X));
		angle.Y = ((angle.Y % (2 * Math.PI)) + 2 * Math.PI) % (2 * Math.PI);
	}

	$: stars = starsRaw
		.map((d) => rotate(d, angle))
		.filter((star) => star.z >= 0 && star.proper != "Sol");

	//Star filter by movement and then projection
	$: lines = linesRaw
		.map(([a, b]) => [rotate(a, angle), rotate(b, angle)])
		.filter(([a, b]) => a.z >= 0 && b.z >= 0);

	$: projection = d3
		.geoOrthographic()
		.scale(width / 2 - margin.v, height / 2 - margin.h)
		.translate([width / 2, height / 2])
		.rotate([angle.Y * (-180 / Math.PI), angle.X * (180 / Math.PI)]);

	$: pathGenerator = d3.geoPath(projection);

	//Tooltip handlers
	let hoveredIndex = -1;
	$: hoveredStar = stars[hoveredIndex] ?? hoveredStar ?? {};
	let cursorPos = { x: 0, y: 0 };
	let tooltipPos = { x: 0, y: 0 };
	let starTooltip;

	async function mouseTooltipHandler(index, evt) {
		let hoveredDot = evt.target;
		if (evt.type == "mouseenter") {
			hoveredIndex = index;
			cursorPos = { x: evt.x, y: evt.y };
			tooltipPos = await computePosition(hoveredDot, starTooltip, {
				strategy: "absolute",
				middleware: [offset(5), autoPlacement()],
			});
		} else if (evt.type == "mouseleave") {
			hoveredIndex = -1;
		}
	}

	$: if (action === "latitude-90") {
		angleAnimation({ X: Math.PI / 2, Y: 0 }, true);
	}

	$: if (action === "latitude-0") {
		angleAnimation({ X: 0, Y: 0 }, true);
	}

	$: if (action === "latitude") {
		angleAnimation({ X: Math.PI / 3, Y: 0 }, true);
	}

	let interval;
	$: {
		if (action === "rotate") {
			if (!interval) {
				interval = d3.interval(() => {
					angle.Y -= 0.005;
				}, 10);
			}
		} else {
			if (interval) {
				interval.stop();
				interval = null;
			}
		}
	}

	$: if (action === "orion") {
		constellation = "Ori";
		angleAnimation({ X: -0.07455171217057399, Y: -1.473016528051261 });
	}

	let limitMag = 7;
	let animatingMag = false;

	$: if (action === "pollution") {
		animatingMag = true;
		magAnimation(limitMag, 4);
	} else {
		magAnimation(limitMag, 7);
		animatingMag = false;
	}

	$: {
		console.log(constellation);
		const curr = constellation
		if (constellation != null) {
			const timeout = setTimeout(() => {
				if(constellation == curr){
					constellation = null;
				}
			}, 3000);
		}
	}

	let svgFocus = false;
</script>

<!-- svg used as canvas for d3 plotting -->
<svg {width} {height} viewBox="0 0 {width} {height}" id="celestial-sphere" class="svg-plot"
	role="figure"
	on:mouseenter={() => svgFocus = true}
	on:mouseleave={() => svgFocus = false}>
	<path
		d={pathGenerator(graticule)}
		fill="none"
		stroke="#444"
		stroke-width="1.5"
		class="graticule-lines"
		class:hide={action === "hide"}
	/>
	<g class="constellation-lines">
		{#each lines as [starA, starB], index (starA.id + "-" + starB.id)}
			<line
				x1={xScale(starA.x)}
				y1={yScale(starA.y)}
				x2={xScale(starB.x)}
				y2={yScale(starB.y)}
				class="constellation"
				class:highlight={constellation === starA.con}
				class:hide={action === "hide" || !(starA.mag <= limitMag && starB.mag <= limitMag)}
			/>
			<!--amarelo: #d9ed2880 -->
			<!--azul: #2f05d990-->
			<!--rosa maluco: #e94b8aB0-->
		{/each}
	</g>
	<path
		d={pathGenerator(sphere)}
		fill="none"
		stroke="#FFF"
		stroke-width="1.5"
	/>"

	<g class="stars">
		{#each stars as star, index (star.id)}
			<circle
				role="tooltip"
				on:mouseenter={(evt) => mouseTooltipHandler(index, evt)}
				on:mouseleave={(evt) => mouseTooltipHandler(index, evt)}
				cx={xScale(star.x)}
				cy={yScale(star.y)}
				r={1.2 * ((5 * (star.mag - 7)) / (-1.45 - 7)) ** 0.9}
				fill={star.rgb}
				class="star"
				class:hide={!(star.mag <= limitMag)}
			/>
		{/each}
	</g>
</svg>
{#if svgFocus}
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
	{/if}

	<dt>Magnitude aparente</dt>
	<dd>{hoveredStar.mag}</dd>

	<dt>Magnitude absoluta</dt>
	<dd>{hoveredStar.absmag}</dd>

	{#if hoveredStar.tem != null}
		<dt>Temperatura</dt>
		<dd>{hoveredStar.tem.toFixed(2)} K</dd>
	{/if}

	{#if hoveredStar.lum != null}
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
{/if}
<style>
	circle {
		transition: 0ms;
		transform-origin: center;
		transform-box: fill-box;
		@starting-style {
			r: 0;
		}
	}

	.graticule-lines {
		opacity: 1;
		transition: opacity 0.5s ease;
	}

	.graticule-lines.hide {
		opacity: 0;
	}

	.star {
		opacity: 1;
		transition: opacity 0.5s ease;
	}

	.star.hide {
		opacity: 0;
	}

	.constellation {
		opacity: 1;
		stroke-width: 1;
		stroke: #aaa;
		transition:
			stroke 0.6s ease,
			stroke-width 0.6s ease,
			opacity 0.5s ease-in-out;
	}

	.constellation.highlight {
		stroke: #e94b8ab0;
		stroke-width: 3;
	}

	.constellation.hide {
		opacity: 0;
	}

	.info {
		margin: 0;

		display: grid;
		grid-template-columns: auto auto;

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
		position: absolute;
		top: 1em;
		left: 1em;
	}
</style>
