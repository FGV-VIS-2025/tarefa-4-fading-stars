<script>
	import * as d3 from "d3";
    export let starsRaw; //Input of all stars, this selector never changes so it needs to be made with all the data
    export let selectedCons; //Output of the component - the max magnitude value selected

    let constelationList = d3.rollups(
        starsRaw, v => v.length, d => d.con
    ).map(
        ([cons, appearances]) => cons
    ).sort();

    let checkedCons = {};
    for(let cons of constelationList){
        checkedCons[cons] = true;
    }
    $: selectedCons = Object.keys(checkedCons).filter(key => checkedCons[key]);

</script>

<div class = "container">
    <fieldset class = "constelations">
        {#each constelationList as cons, index}
        <label>
            <input type = "checkbox" bind:checked = {checkedCons[cons]}/>
            {cons}
        </label>
        {/each}
    </fieldset>
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
    justify-content: space-between;
    flex-wrap: wrap;
}
</style>
