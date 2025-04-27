<script>
	import consMap from "$lib/data/constellations-map.json";
	import * as d3 from "d3";
    export let starsRaw; //Input of all stars, this selector never changes so it needs to be made with all the data
    export let selectedCons; //Output of the component - the filtered cons
    export let consPosition; //Output of the component - the selected cons location

    //To hold what the user types in input
    let userQuery = "";
    //Initial constellation list
    let constellationList = d3.rollups(
        starsRaw, v => v.length, d => d.con
    ).map(
        ([cons, appearances]) => cons
    ).sort();
    //object to mark whenever a constellation is checked
    let checkedCons = {};
    //map of constellation -> angle to focus
    let consAngle = {};
    function consFocus(cons){
        //star with lowest magnitude
        let stars = starsRaw.filter(star => star.con == cons);
        let x_proj = stars.map(star => star.x_proj).reduce((a, b) => a + b) / stars.length,
            y_proj = stars.map(star => star.y_proj).reduce((a, b) => a + b) / stars.length,
            z_proj = stars.map(star => star.z_proj).reduce((a, b) => a + b) / stars.length;
        let X, Y;
        if(star.z_proj > 0){
            Y = Math.sign(x_proj) * Math.atan(Math.abs(x_proj/z_proj));
        } else {
            Y = Math.sign(x_proj) * (Math.atan(Math.abs(z_proj/x_proj)) + Math.PI/2);
        }
        X = Math.sign(y_proj) * Math.atan(Math.abs(y_proj/z_proj));
        return {X: X, Y: Y};
    }

    for(let cons of constellationList){
        checkedCons[cons] = true;
        consAngle[cons] = consFocus(cons);
    }

    let queriedCons;
    $: queriedCons = constellationList.filter(
        cons => ((cons != null) && (consMap[cons].toLowerCase().includes(userQuery.toLowerCase())))
    );

    $: selectedCons = Object.keys(checkedCons).filter(key => checkedCons[key]);


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
            <span class="showButton" on:click={evt => consPosition = consAngle[cons]}>(focar)</span>
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
