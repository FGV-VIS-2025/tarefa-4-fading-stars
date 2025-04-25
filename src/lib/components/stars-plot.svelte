<script>
	import * as d3 from "d3";

	import stars from "$lib/data/stars-data.json";

	let width = 600;
	let height = 600;

	$: xScale = d3.scaleLinear().range([0, width]);
	$: yScale = d3.scaleLinear().range([0, height]);
</script>

<svg {width} {height} viewBox="-600 -600 {width * 2} {height * 2}">
	<g class="stars">
		{#each stars as star}
			<circle
				cx={xScale(star.x_proj)}
				cy={yScale(star.y_proj)}
				r="1"
				fill="white"
			/>
		{/each}
	</g>
</svg>
