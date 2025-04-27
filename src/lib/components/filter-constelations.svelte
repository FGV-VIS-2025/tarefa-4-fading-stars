<script>
	import consMap from "$lib/data/constellations-map.json";
	import * as d3 from "d3";
    export let starsRaw; //Input of all stars, this selector never changes so it needs to be made with all the data
    export let selectedCons; //Output of the component - the filtered cons
    export let consPosition; //Output of the component - the selected cons location

    let userQuery = "";

    let constellationList = d3.rollups(
        starsRaw, v => v.length, d => d.con
    ).map(
        ([cons, appearances]) => cons
    ).sort();
    let checkedCons = {};
    for(let cons of constellationList){
        checkedCons[cons] = true;
    }

    let queriedCons;
    $: queriedCons = constellationList.filter(
        cons => ((cons != null) && (cons.toLowerCase().includes(userQuery.toLowerCase())))
    );

    $: selectedCons = Object.keys(checkedCons).filter(key => checkedCons[key]);

    //TODO: Computar antes e fazer de um jeito inteligente
    function consFocus(evt, cons){
        for(let star of starsRaw){
            if(star.con == cons){
                console.log(star);
            }
        }
    }

</script>

<div class = "container">
    <h3>Filtre as constelações visíveis.</h3>
    <label for="consSearch">Busque por uma constelação!</label>
    <input type="text" id = "consSearch" bind:value = {userQuery}/>
    <fieldset class="constelations">
        {#each queriedCons as cons, index}
        <div class = consName>
            <label>
                <input name = "marker{cons}" type="checkbox" bind:checked={checkedCons[cons]}/>
                {consMap[cons]}
            </label>
            <span class="showButton" on:click={evt => consFocus(evt, cons)}>(focar)</span>
        </div>
        {/each}
    </fieldset>
    <p>Valor digitado na caixa de texto: {userQuery}</p>
</div>


<style>
.container{
    border-style: solid;
    border-radius: 6px;
    border-width: 2px;

    padding: 20px;
}

h3{
    margin: auto;
    text-align: justify;
}

.constelations{
    display: flex;
    flex-direction: column;
    max-height: 15ch;
    width: 21ch;
    overflow-y: auto;
}

.consName:hover .showButton{
    display: inline;
}

.showButton{
    display: none;
    color: #e6e6e6;
    font-size: 80%;
    text-decoration: underline;
}

</style>
