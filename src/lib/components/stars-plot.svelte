<script>
	import * as d3 from "d3";
	import { onMount } from "svelte";

	import starsRaw from "$lib/data/stars-data.json";

	let width = 600;
	let height = 600;

	let angle = { X: 0, Y: 0 };
	let dragSpeed = 0.002;

	const graticule = d3.geoGraticule10();

	$: xScale = d3.scaleLinear().range([0, width]);
	$: yScale = d3.scaleLinear().range([0, height]);

	onMount(() => {
		const dragFun = d3.drag().on("drag", (event) => {
			angle.X -= event.dy * dragSpeed;
			angle.Y -= event.dx * dragSpeed;
		});

		d3.select("body").call(dragFun);
	});

	function rotate(d, angle) {
		let x, y, z;
		x = d.x_proj * Math.cos(angle.Y) - d.z_proj * Math.sin(angle.Y);
		z = d.x_proj * Math.sin(angle.Y) + d.z_proj * Math.cos(angle.Y);
		y = d.y_proj * Math.cos(angle.X) - z * Math.sin(angle.X);
		z = d.y_proj * Math.sin(angle.X) + z * Math.cos(angle.X);

		return { ...d, x: x, y: y, z: z };
	}

	$: stars = starsRaw
		.map((d) => rotate(d, angle))
		.filter(
			(star) =>
				star.z >= 0 &&
				(star.proper == "Polaris" || star.proper == "Sol"),
		);
	// $: console.log(angle);
	$: projection = d3
		.geoOrthographic()
		.scale(width, height)
		.translate([0, 0])
		.rotate([angle.Y * (-180 / Math.PI), angle.X * (180 / Math.PI)]);
	$: pathGenerator = d3.geoPath(projection);
</script>

<input type="number" bind:value={angle.X} />
<input type="number" bind:value={angle.Y} />

<svg {width} {height} viewBox="-{width} -{height} {width * 2} {height * 2}">
	<path
		d={pathGenerator(graticule)}
		fill="none"
		stroke="#777"
		stroke-width="1.5"
	/>
	<g class="stars">
		{#each stars as star}
			<circle
				cx={xScale(star.x)}
				cy={yScale(star.y)}
				r={2 * ((5 * (star.mag - 7)) / (-1.45 - 7)) ** 0.5}
				fill={star.rgb}
			/>
		{/each}
	</g>
</svg>
