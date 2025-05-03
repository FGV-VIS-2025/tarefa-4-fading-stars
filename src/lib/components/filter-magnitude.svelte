<script>
    import Help from "$lib/components/Help.svelte";
    import { onMount } from "svelte";

    import * as d3 from "d3";
    export let starsRaw; //Input of all stars, this slider never changes so it needs to be made with all the data
    export let maxMagnitude; //Output of the component - the max magnitude value selected
    export let percentage;
    export let userCoordinates;

    let canvas;
    let canvasContext;
    let img;
    onMount(() => {
        img = new Image();
        img.onload = () => console.log("oiii");
        img.src = "/world2024.png";
        canvas.width = 100;
        canvas.height = 100;
        canvasContext = canvas.getContext("2d");
	});

	function getCoordinateMag(lat, lon){
        console.log(lat, lon);
        let x, y;
        if(-65 <= lat && lat <= 75 && -180 <= lon && lon <= 180){
            x = Math.floor((((lon*1) + 180) / 360) * img.width);
            y = Math.floor(((75 - (lat*1)) / 140) * img.height);
        }
        console.log(x, y);
        canvasContext.drawImage(img, -x, -y);
        const pixel = canvasContext.getImageData(0, 0, 1, 1);
        console.log(pixel);
//        console.log(`RGB: (${pixel[0]}, ${pixel[1]}, ${pixel[2]})`);

//         def latlon_para_pixel(lat, lon, largura=43200, altura=16800):
// ...     if not (-65 <= lat <= 75):
// ...         ...
// ...     if not (-180 <= lon <= 180):
// ...         ...
// ...
// ...     x = int((lon + 180) / 360 * largura)
// ...     y = int((75 - lat) / 140 * altura)
// ...     return x, y
	};

    let maxDataMagnitude, minDataMagnitude;
    maxDataMagnitude = d3.max(starsRaw.map((star) => star.mag));
    minDataMagnitude = d3.min(starsRaw.slice(1).map((star) => star.mag));

    let userBarInput = 200;
    let linScale = d3
        .scaleLinear()
        .domain([minDataMagnitude, maxDataMagnitude])
        .range([0, 200]);
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
                max="200"
                bind:value={userBarInput}
            />
            <div class="minmax">
                <span>{minDataMagnitude} m</span><span
                    >{maxDataMagnitude} m</span
                >
            </div>
            <span
                >Magnitude máxima selecionada: {maxMagnitude.toFixed(2)} m</span
            ><br />
            <span
                >Porcentagem de estrelas exibidas: {(percentage * 100).toFixed(
                    1,
                )}%</span
            >
            <button on:click = {evt => getCoordinateMag(userCoordinates.lat, userCoordinates.lon)}>aperta</button>
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
</div>

<canvas bind:this={canvas}></canvas>

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
