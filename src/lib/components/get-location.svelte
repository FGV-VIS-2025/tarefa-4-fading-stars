<script>
    import Help from "$lib/components/Help.svelte";

    //Output of the component - a lat lon pair from the place the user searched and searched
    export let coordinates = { lat: 0, lon: 0 };
    export let action = null;

    //To get what the user typed
    let userInput;
    //To manage search success
    let interacted = false;
    let searching = false;
    let successfulRequest = false;
    let successfulSearch = false;
    //To manage user choice on results
    let searchResults = [];
    let selectedResult;

    function searchPlaces(evt) {
        evt.preventDefault();
        interacted = true;
        searching = true;
        let link = `https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(userInput)}&layer=adress&featureType=city&addressdetails=1&limit=9`;
        fetch(link)
            .then((response) => response.json())
            .then((data) => {
                searching = false;
                successfulRequest = true;
                if (data.length > 0) {
                    console.log(data);
                    let possible_fields = ["city", "municipality", "town", "village"]
                    //idk if this will stay here...
                    for (let result of data) {
                        result.display_text = result.display_name;
                        for (let field of possible_fields){
                            if (result.address[field] != null) {
                                result.display_text = result.address[field];
                                if(result.address.state != null){result.display_text += " - " + result.address.state;}
                                result.display_text += " - " + result.address.country;
                                break;
                            }
                        }
                    }
                    successfulSearch = true;
                    searchResults = data;
                    selectedResult = 0;
                } else {
                    successfulSearch = false;
                    searchResults = [];
                    selectedResults = -1;
                }
            })
            .catch((error) => {
                console.error("Error while searching for places:", error);
                successfulRequest = false;
                successfulSearch = false;
                searchResults = [];
                selectedResult = -1;
            });
    }

    function onResultClick(evt, index) {
        selectedResult = index;
    }

    $: {
        if (searchResults.length > 0) {
            coordinates.lat = searchResults[selectedResult].lat * 1;
            coordinates.lon = searchResults[selectedResult].lon * 1;
        } else {
            coordinates = { lat: 0, lon: 0 };
        }
    }
</script>

<div class="container">
    <h3>Busque por uma cidade.</h3>
    <form on:submit={searchPlaces}>
        <label for="cityInput"
            >Escreva o nome de uma cidade e aperte em buscar.</label
        >
        <div class="searchBar">
            <input id="cityInput" type="text" bind:value={userInput} required />
            <button type="submit">Buscar</button>
        </div>
    </form>

    {#if interacted == false}
        <p>Nenhuma busca foi feita.</p>
    {:else if searching == true}
        <p>Pesquisando cidades...</p>
    {:else if successfulRequest == false}
        <p class="erro">Erro na busca. Tente novamente.</p>
    {:else if successfulSearch == false}
        <p class="erro">
            Busca sem resultados. Dê preferência por digitar nomes completos de
            cidade ao invés de apenas um pedaço do nome.
        </p>
    {:else}
        <div class="results">
            {#each searchResults as result, index}
                {#if selectedResult == index}
                    <div
                        class="searchResult selected"
                        on:click={(evt) => onResultClick(evt, index)}
                    >
                        {result.display_text}
                    </div>
                {:else}
                    <div
                        class="searchResult"
                        on:click={(evt) => onResultClick(evt, index)}
                    >
                        {result.display_text}
                    </div>
                {/if}
            {/each}
        </div>
    {/if}
    {#if successfulSearch}
        <p>A cidade selecionada está localizada na latitude {coordinates.lat.toFixed(2)}° e na longitude {coordinates.lon.toFixed(2)}°.</p>
        <p on:mouseenter={evt => action = "rotate"}
              on:mouseleave={evt => action = null}
              class="highlight"
        >Veja a movimentação do esfera celeste ao longo de um dia.</p>
    {/if}
	<Help alignTop={false}>
        <p>
            A localização de um observador interfere na observação do esfera celeste: existem estrelas que
            só são visíveis apenas do hemisfério sul, bem como estrelas que só são visíveis do hemisfério norte.
        </p>
        <p>
            Como a Terra rotaciona em torno de um eixo um pouco deslocado em comparação com os polos, a
            latitude da localização é o fator que determina quais estrelas nunca poderão ser vistas de um determinado
            local, enquanto a longitude influencia em qual será o momento do dia em que determinado pedaço do esfera
            celeste estará acima de um observador. Evidentemente, como a luz do sol impede a observação das estrelas,
            na prática uma longitude diferente interferirá em que época do ano determinadas constelações serão vistas.
        </p>
        <p>
            Essa ferramenta permite com que uma cidade seja pesquisada, e posiciona o esfera celeste para
            coincidir com o pedaço visível da latitude dessa cidade. Após selecionar uma cidade, é fornecida
            a opção de girar a esfera em torno do eixo, permitindo ver como o céu fica em diferentes partes
            do dia, ignorando a luz solar.
        </p>
    </Help>

</div>

<style>
    /*Coloquei borda aqui pra não me perder na estilização dos outros componentes.  Depois é só tirar*/
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

    .searchBar {
        margin: 5px 0;

        display: grid;
        grid-template-columns: auto 7ch;
        gap: 1ch;
    }

    .erro {
        color: #ff4c4c;
    }

    .results {
        display: flex;
        flex-wrap: wrap;
        justify-content: center;
    }

    .searchResult {
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

    .selected {
        border-color: #2f05d9b6;
        background-color: #2f05d93a;
    }

    .highlight {
		background-color: #f1f1f1;
		color: #000;
		display: inline-block;
		margin: 3px 0;
		padding: 3px;
		font-weight: bold;
		cursor: default;
	}

</style>
