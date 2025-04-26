<script>
	import * as d3 from "d3";

	import starsRaw from "$lib/data/stars-data.json";

	let width = 600;
	let height = 600;

	let angle = { X: 0, Y: 0 };

	$: xScale = d3.scaleLinear().range([0, width]);
	$: yScale = d3.scaleLinear().range([0, height]);

	function rotate(d, angle) {
		let x = d.x_proj * Math.cos(angle.Y) - d.z_proj * Math.sin(angle.Y);
		let z = d.x_proj * Math.sin(angle.Y) + d.z_proj * Math.cos(angle.Y);
		let y = d.y_proj * Math.cos(angle.X) - z * Math.sin(angle.X);
		z = d.y_proj * Math.sin(angle.X) + z * Math.cos(angle.X);

		return {...d, x: x, y: y, z: z};
	}

	$: stars = starsRaw.map(d => rotate(d, angle)).filter((star) => star.z >= 0);

	$: console.log(angle);
</script>

<input type="number" bind:value={angle.X} />
<input type="number" bind:value={angle.Y} />

<svg {width} {height} viewBox="-600 -600 {width * 2} {height * 2}">
	<g class="stars">
		{#each stars as star}
			<circle
				cx={xScale(star.x)}
				cy={yScale(star.y)}
				r="1"
				fill="white"
			/>
		{/each}
	</g>
</svg>
