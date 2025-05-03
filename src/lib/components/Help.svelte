<script>
    let showHelp = false;
</script>

<button
    class="help-button"
    class:selected={showHelp}
    on:mouseenter={() => (showHelp = true)}
    on:mouseleave={() => (showHelp = false)}
>
    <!-- on:click={() => showHelp = !showHelp} -->
    ?
</button>



{#if showHelp}
    <div class="overlay"></div>
    <div class="help-div">
        <slot></slot>
    </div>
{/if}

<style>
    .help-button {
        cursor: pointer;
        width: 2.5ch;
        height: 2.5ch;

        border-style: solid;
        border-radius: 6px;
        border-width: 2px;

        text-align: center;
        align-items: center;
        justify-content: center;
        align-self: end;

        padding: 0;

        position: absolute;
        bottom: 2ch;
        right: 2ch;
        font-weight: bolder;

        transition:
            color 0.3s ease,
            background-color 0.3s ease;
    }

    .help-button.selected {
        z-index: 999;
        background-color: antiquewhite;
        color: black;
    }

    .help-div {
        position: absolute;

        width: 80%;
        left: 105%;
        top: 0;

        border-style: solid;
        border-radius: 6px;
        border-width: 2px;
        padding: 2ch;

        background-color: var(--accent-black);
        z-index: 2;

        transition:
            display 0.3s ease,
            filter 0.3s ease,
            opacity 0.3s ease;
    }

    .overlay {
        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        backdrop-filter: blur(2px);
        z-index: 1;
        transition: opacity 0.3s ease;
    }
</style>
