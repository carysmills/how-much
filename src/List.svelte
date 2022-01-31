<script>
    import { onDestroy } from "svelte";
    import data from "./data/cost-filtered.json";
    import { selected } from "./stores.js";

    let selectedItems;

    function formatTitle(title) {
        const indexOfBracket = title.indexOf("(");
        return title.slice(0, indexOfBracket);
    }

    selected.subscribe((value) => {
        selectedItems = value;
    });

    function updateItem(item) {
        selected.update((n) => {
            const itemExists = n.includes(item);
            if (itemExists) {
                const newVal = n.filter((x) => x != item);
                selectedItems = newVal;
                return selectedItems;
            } else {
                const newVal = [...n, item].sort();
                selectedItems = newVal;
                return selectedItems;
            }
        });
    }

    const unsubscribe = selected.subscribe((value) => {
        selectedItems = value;
    });

    onDestroy(unsubscribe);
</script>

<div class="card">
    <div class="card-header">
        <h2 class="card-title">Select groceries</h2>
    </div>

    <form action="">
        {#each Object.keys(data) as item}
            <div class="card-text">
                <label class="form-control" for={item}>
                    <input
                        type="checkbox"
                        checked={selectedItems.includes(item)}
                        id={item}
                        name={item}
                        on:change={(item) => {
                            updateItem(item.target.name);
                        }}
                    />
                    {formatTitle(item)}
                </label>
            </div>
        {/each}
    </form>
</div>

<style>
    * {
        font-family: "Courier New", Courier, monospace;
    }

    :root {
        --form-control-color: #373e98;
    }

    form {
        -moz-column-count: 2;
        -webkit-column-count: 2;
        column-count: 2;
        margin-top: 30px;
    }

    .form-control {
        display: grid;
        grid-template-columns: 1em minmax(auto, 200px);
        gap: 5px;
    }

    input[type="checkbox"] {
        /* Add if not using autoprefixer */
        -webkit-appearance: none;
        /* Remove most all native input styles */
        appearance: none;
        /* For iOS < 15 */
        background-color: var(--form-background);
        /* Not removed via appearance */
        margin: 0;
        width: 1.15em;
        height: 1.15em;
        border: 0.15em solid currentColor;
        border-radius: 0.15em;
        transform: translateY(6px);
        display: grid;
        place-content: center;
    }

    input[type="checkbox"]::before {
        content: "";
        width: 0.65em;
        height: 0.65em;
        clip-path: polygon(14% 44%, 0 65%, 50% 100%, 100% 16%, 80% 0%, 43% 62%);
        -webkit-clip-path: polygon(
            14% 44%,
            0 65%,
            50% 100%,
            100% 16%,
            80% 0%,
            43% 62%
        );
        transform: scale(0);
        transform-origin: bottom left;
        transition: 120ms transform ease-in-out;
        box-shadow: inset 1em 1em var(--form-control-color);
        /* Windows High Contrast Mode */
        background-color: CanvasText;
    }

    input[type="checkbox"]:checked::before {
        transform: scale(1);
    }

    input[type="checkbox"]:focus {
        outline: max(2px, 0.15px) solid darkslategray;
    }

    .card {
        transform: rotate(-0.002turn);
        background-color: white;
        background: repeating-linear-gradient(
            white,
            white 25px,
            #9198e5 26px,
            #9198e5 27px
        );
        background-position-y: 34px;
        margin-right: 15px;
        box-shadow: rgba(100, 100, 111, 0.1) 0px 7px 20px 0px;
        border: 1px solid whitesmoke;
        padding-bottom: 20px;
        height: max-content;
        min-width: min-content;
    }
    .card-header {
        background: linear-gradient(white, white 33px, pink 35px, pink 36px);
        height: 36px;
    }
    .card-title {
        position: relative;
        left: 10px;
        top: 2px;
        font-size: 30px;
    }
    .card-text {
        font-size: 18px;
        margin-left: 20px;
        line-height: 27px;
        -webkit-column-break-inside: avoid;
        column-break-inside: avoid;
        page-break-inside: avoid;
        break-inside: avoid;
    }

    @media only screen and (max-width: 768px) {
        .card {
            margin-right: 0;
            transform: none;
        }

        .card-text {
            font-size: 14px;
        }
    }
</style>
