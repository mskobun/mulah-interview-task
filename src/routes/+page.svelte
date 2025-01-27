<script>
    import { onMount } from "svelte";
    // Import chota for basic styling
    import "chota";
    import Papa from "papaparse";

    let info = $state("");
    let error = $state("");
    let tableData = $state([]);
    // Map of indices to values
    let valueMap = $derived(
        tableData.reduce((acc, row) => {
            acc[row["Index #"]] = row["Value"];
            return acc;
        }, {}),
    );
    let headers = $state([]);
    let fileInput;

    function parseTable(file_or_url) {
        Papa.parse(file_or_url, {
            download: true,
            header: true,
            complete: function (results) {
                if (results.data.length > 0) {
                    headers = Object.keys(results.data[0]);
                }
                if (
                    !headers.includes("Index #") ||
                    !headers.includes("Value")
                ) {
                    error = "CSV file does not contain the required columns";
                } else {
                    error = "";
                    // Convert all values to integers
                    results.data.map((row) => {
                        row["Value"] = parseInt(row["Value"]);
                    });
                    tableData = results.data;
                }
            },
            error: function (err) {
                error = "Error parsing CSV file";
                console.error(err);
            },
        });
    }

    function handleFileSelect(event) {
        const file = event.target.files[0];
        if (file) {
            parseTable(file);
        }
        info = "";
    }
    // Load default table on page load
    onMount(() => {
        parseTable("/Table_Input.csv");
        info =
            "Loaded default table. Use the file input to load your own table.";
    });
</script>

<main class="container">
    <h1>Table Viewer</h1>

    {#if info}
        <p class="bg-light">{info}</p>
    {/if}
    <div class="file-input">
        <input
            type="file"
            accept=".csv"
            bind:this={fileInput}
            onchange={handleFileSelect}
        />
    </div>

    {#if error}
        <p class="bg-error">{error}</p>
    {:else if tableData.length > 0}
        <h2>Table 1</h2>
        <table>
            <thead>
                <tr>
                    {#each headers as header}
                        <th>{header}</th>
                    {/each}
                </tr>
            </thead>
            <tbody>
                {#each tableData as row}
                    <tr>
                        {#each headers as header}
                            <td>{row[header]}</td>
                        {/each}
                    </tr>
                {/each}
            </tbody>
        </table>
        <h2>Table 2</h2>
        <table>
            <thead>
                <tr>
                    <th>Category</th>
                    <th>Value</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>Alpha</td>
                    <td>{valueMap["A5"] + valueMap["A20"]}</td>
                </tr>
                <tr>
                    <td>Beta</td>
                    <td>{valueMap["A15"] / valueMap["A7"]}</td>
                </tr>
                <tr>
                    <td>Charlie</td>
                    <td>{valueMap["A13"] * valueMap["A12"]}</td>
                </tr>
            </tbody>
        </table>
    {/if}
</main>

<style>
    table {
        max-width: 200px;
    }

    td,
    th {
        text-align: center;
    }
    .container {
        max-width: 720px;
    }
</style>
