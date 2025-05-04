<script>
    import Help from "$lib/components/Help.svelte";
    import { onMount } from "svelte";

    import * as d3 from "d3";
    export let starsRaw; //Input of all stars, this slider never changes so it needs to be made with all the data
    export let maxMagnitude; //Output of the component - the max magnitude value selected
    export let percentage;
    export let userCoordinates;

    const rgbMap = {
        "#FFFFFF": 3.799093644700977,
        "#BFBFBF": 4.2820872887782375,
        "#ED3833": 4.7338747438940665,
        "#9A2421": 5.14477892481593,
        "#D87635": 5.507673538161097,
        "#A25827": 5.812163681385028,
        "#CEC842": 6.0641111172184825,
        "#908C2E": 6.2535872569129,
        "#53B12E": 6.389567967218516,
        "#316A1B": 6.479153564151049,
        "#013EDC": 6.536806275297105,
        "#012584": 6.574322752292328,
        "#464646": 6.59280597486233,
        "#232323": 6.620187747545812,
        "#000000": 6.624711410308956,
    };

    let canvas;
    let canvasContext;
    let img;
    let imgMounted = false;
    onMount(() => {
        //creates image object and associated canvas
        img = new Image();
        img.onload = () => {
            canvas.width = img.width;
            canvas.height = img.height;
            canvasContext = canvas.getContext("2d");
            canvasContext.drawImage(img, 0, 0);
            imgMounted = true;
        }
        img.src = "/world2024_low3.png";

	});

    function isOnRange(lat, lon){
        return ((lat != 0) || (lon != 0)) && (-65 <= lat) &&
        (lat <= 75) && (-180 <= lon) && (lon <= 180);
    }

	function getCoordinateMag(lat, lon){
        let x, y;
        x = Math.floor(((lon + 180) / 360) * img.width);
        y = Math.floor(((75 - lat) / 140) * img.height);
        const pixel = canvasContext.getImageData(x, y, 1, 1).data;
        let rgb = pixel[2] | (pixel[1] << 8) | (pixel[0] << 16);
        rgb = '#' + rgb.toString(16).padStart(6, '0').toUpperCase();

        let newUBI = linScale(rgbMap[rgb]);
        let originalUBI = userBarInput;

        const duration = Math.abs(rgbMap[rgb] - maxMagnitude) * 300;
		const start = Date.now();
		const transitionTimer = d3.timer(() => {
			const elapsed = Date.now() - start;
			const t = Math.min(1, elapsed / duration);
            userBarInput = (newUBI - originalUBI)*t + originalUBI;
			if (t >= 1) {
				transitionTimer.stop();
			}
		});
	};

    let maxDataMagnitude, minDataMagnitude;
    maxDataMagnitude = d3.max(starsRaw.map((star) => star.mag));
    minDataMagnitude = d3.min(starsRaw.slice(1).map((star) => star.mag));

    let userBarInput = 300;
    let linScale = d3
        .scaleLinear()
        .domain([minDataMagnitude, maxDataMagnitude])
        .range([0, 300]);
    $: maxMagnitude = linScale.invert(userBarInput);
</script>

<div class="container">
    <h3>Filtre por magnitude.</h3>

    <div class="slider-container">
        <div class="slider">
            <label for="slider">Magnitude máxima:</label>
            <input
                type="range"
                id="slider"
                name="slider"
                min="0"
                max="300"
                bind:value={userBarInput}
            />
            <div class="minmax">
                <span>{minDataMagnitude} m</span><span
                    >{maxDataMagnitude} m</span
                >
            </div>
            {#if imgMounted && isOnRange(userCoordinates.lat, userCoordinates.lon)}
                <button on:click = {evt => getCoordinateMag(userCoordinates.lat, userCoordinates.lon)}>
                Sincronizar com a cidade selecionada</button>
            {/if}
            <br/><span>Magnitude máxima selecionada: {maxMagnitude.toFixed(2)} m</span><br/>
            <span>Porcentagem de estrelas exibidas: {(percentage * 100).toFixed(1)}%</span>
            <!--TODO: improve label or move button to other spot-->
        </div>
    </div>

    <Help>
        <p>
            Cada estrela tem associado a si um valor de magnitude aparente, que
            diz o quão brilhante uma estrela aparenta ser quando vista da terra.
            Estrelas mais brilhantes (com magnitude <i>menor</i>), naturalmente,
            são visiveis mesmo com outras fontes de luz por perto, enquanto
            estrelas menos brilhantes (com magnitude <i>maior</i>) não são
            vistas quando há outras fontes de luz interferindo.
        </p>
        <p>
            Esse slider permite mostrar apenas as estrelas mais brilhantes,
            simulando diferentes níveis de poluição visual interferindo na
            observação do globo celeste.
        </p>
    </Help>
    <canvas bind:this={canvas} hidden=true></canvas>
</div>


<style>
    .container {
        border-style: solid;
        border-radius: 6px;
        border-width: 2px;

        margin-bottom: 20px;

        padding: 2ch;
        position: relative;
    }

    h3 {
        margin: auto;
        text-align: justify;
    }

    #slider {
        width: 100%;
    }
    .minmax {
        display: flex;
        justify-content: space-between;
        font-size: 70%;
    }
</style>
