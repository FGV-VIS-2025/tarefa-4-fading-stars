<script>
    export let coordinates = {lat: 0, lon: 0};
    //To get what the user typed
    let userInput;
    //To manage search success
    let successfulRequest = false;
    let successfulSearch = false;
    //To manage user choice on results
    let searchResults = [];
    let selectedResult;

    function searchPlaces(evt){
        evt.preventDefault();
        let link = `https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(userInput)}&layer=adress&featureType=city&addressdetails=1`
        fetch(link)
            .then(response => response.json())
            .then(data => {
                successfulRequest = true;
                if (data.length > 0) {
                    console.log(data);
                    //idk if this will stay here...
                    for(let result of data){
                        if(result.address.city != null){
                            result.display_text = result.address.city + " - " + result.address.country;
                        } else if(result.address.municipality != null){
                            result.display_text = result.address.municipality + " - " + result.address.country;
                        } else if(result.address.town != null){
                            result.display_text = result.address.town + " - " + result.address.country;
                        } else {
                            result.display_text = result.display_name;
                        }
                    }
                    successfulSearch = true;//
                    searchResults = data;
                    selectedResult = 0;
                } else {
                    successfulSearch = false;
                    searchResults = [];
                }
              })
          .catch(error => {
            console.error('Error while searching for places:', error);
            successfulRequest = false;
            successfulSearch = false;
        });
    }

    function onResultClick(evt, index){
        console.log("clicou no ");
        console.log(index);
        selectedResult = index;
    }

    $: {
        if(searchResults.length > 0){
            coordinates.lat = searchResults[selectedResult].lat;
            coordinates.lon = searchResults[selectedResult].lon;
        } else {coordinates = {lat: 0, lon: 0}}
        console.log(coordinates);
    }
</script>
<div class = "container">
    <h3>Busque por uma cidade.</h3>

    <form on:submit={searchPlaces}>
        <input type="text" bind:value={userInput} required>
        <button type="submit">Buscar</button>
    </form>

    {#if successfulRequest == false}
        <p class = "erro">Erro na busca. Tente novamente.</p>
    {:else if successfulSearch == false}
        <p class = "erro">Busca sem resultados. Dê preferência por digitar nomes
        completos de cidade ao invés de apenas um pedaço do nome.</p>
    {:else}
        <div class = "results">
        {#each searchResults as result, index}
        {#if selectedResult == index}
            <div class="searchResult selected" on:click={evt => onResultClick(evt, index)}>
                {result.display_text}
            </div>
        {:else}
            <div class="searchResult" on:click={evt => onResultClick(evt, index)}>
                {result.display_text}
            </div>
        {/if}
        {/each}
        </div>
    {/if}

    <p>Texto digitado na caixa de texto: {userInput}</p>
</div>

<style>
/*Coloquei borda aqui pra não me perder na estilização dos outros componentes.  Depois é só tirar*/
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

form {
    margin: 5px 0;

    display: grid;
    grid-template-columns: auto 6ch;
    gap: 1ch;
}

.erro{
    color: #ff4c4c
}

.results {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
}

.searchResult{
    max-width: 20ch;

    margin: 4px;

    border-style: solid;
    border-color: #777777;
    border-width: 2px;
    border-radius: 4px;

    padding: 6px;

    font-size: 75%;
    color: #e6e6e6;
    text-transform: uppercase;
    font-weight: bold;
}

.selected{
    border-color: #2f05d9B6;
    background-color: #2f05d93a;
}
</style>
