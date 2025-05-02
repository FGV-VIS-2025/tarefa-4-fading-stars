<script>
	import * as d3 from "d3";
    export let starsRaw; //Input of all stars, this slider never changes so it needs to be made with all the data
    export let maxMagnitude; //Output of the component - the max magnitude value selected
    export let percentage;

    let maxDataMagnitude, minDataMagnitude;
    maxDataMagnitude = d3.max(starsRaw.map(star => star.mag));
    minDataMagnitude = d3.min(starsRaw.slice(1).map(star => star.mag));

    let userBarInput = 200;
    let linScale = d3.scaleLinear().domain([minDataMagnitude, maxDataMagnitude]).range([0,200]);
    $: maxMagnitude = linScale.invert(userBarInput);

    let hoveredButton = false;
</script>

<div class = "container">
    <h3>Filtre por magnitude.</h3>

    <div class="slider-container">
        <div class="slider">
            <label for="slider">Magnitude máxima:</label>
            <input type="range" id="slider" name="slider" min=0 max=200 bind:value={userBarInput}/>
            <div class="minmax"><span>{minDataMagnitude} m</span><span>{maxDataMagnitude} m</span></div>
            <span>Magnitude máxima selecionada: {maxMagnitude.toFixed(2)} m</span><br>
            <span>Porcentagem de estrelas exibidas: {(percentage * 100).toFixed(1)}%</span>
        </div>
    </div>
    <div class="help-box"
		 on:mouseenter={evt => hoveredButton = true}
		 on:mouseleave={evt => hoveredButton = false}
	>?</div>
	{#if hoveredButton}
    <div class="help-hover">
        <p>Cada estrela tem associado a si um valor de magnitude aparente, que diz
        o quão brilhante uma estrela aparenta ser quando vista da terra. Estrelas mais
        brilhantes (com magnitude <i>menor</i>), naturalmente, são visiveis mesmo
        com outras fontes de luz por perto, enquanto estrelas menos brilhantes
        (com magnitude <i>maior</i>) não são vistas quando há outras fontes de luz
        interferindo. </p>
        <p>Esse slider permite mostrar apenas as estrelas mais brilhantes,
        simulando diferentes níveis de poluição visual interferindo na observação do
        globo celeste.</p>
    </div>
    {/if}
</div>


<style>
.container{
    border-style: solid;
    border-radius: 6px;
    border-width: 2px;

    margin-bottom: 20px;

    padding: 2ch;
    position: relative;

}

h3{
    margin: auto;
    text-align: justify;
}

#slider{
	width: 100%;
}
.minmax{
    display: flex;
    justify-content:space-between;
    font-size: 70%;
}


.help-box {
    width: 2.5ch;
    height: 2.5ch;

    border-style: solid;
    border-radius: 6px;
    border-width: 2px;

    text-align: center;
    align-self: end;

    padding: 1px;

    position: absolute;
    bottom: 2ch;
    right: 2ch;
}

.help-hover {
    position: absolute;
    top: 10%;
    left: 10%;
    width: 80%;

    font-size: 80%;
    text-align: justify;

    border-style: solid;
    border-radius: 6px;
    border-width: 2px;
    padding: 5px;

    background-color: var(--accent-black);
    z-index: 2;
}


</style>
