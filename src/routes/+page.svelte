<script>
	import StarsPlot from "$lib/components/stars-plot.svelte";
</script>

<div class="all">
	<div class="filters">
		<h2>Aqui entram os filtros</h2>
	</div>

	<div class="viz">
		<h2>Aqui entra a viz</h2>
		<StarsPlot />
	</div>
</div>

<style>
	.all {
		display: grid;
		grid-template-columns: 40% 45%;
	}
</style>
