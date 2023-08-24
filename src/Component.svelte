<script>
    import {getContext, onMount} from 'svelte';

    const { API, notificationStore } = getContext('sdk');

    export let componentTitle = "";
    export let dataProvider;
    export let labelColumn;
    export let orderColumn;
    export let sortOrder;
    export let showOrder = true;
    let dragIndex = null;
    let dropIndex = null;
    let tableStatuses = [];
    const statusTableId = dataProvider?.datasource.tableId;
    $: reactiveTableStatuses = tableStatuses;

    async function fetchTables() {
        try {
            tableStatuses = await API.fetchTableData(statusTableId);
            if (tableStatuses.length > 0) {
                tableStatuses.sort((a, b) => b[orderColumn] - a[orderColumn]);
            }
        } catch (error) {
            console.log("SOME ERROR");
        }
    }

    function handleDragStart(event, index) {
        event.dataTransfer.effectAllowed = 'move';
        dragIndex = index;
    }
    function handleDragOver(event, index) {
        event.preventDefault();
        if (index === dragIndex) return;
        dropIndex = index;
    }

    async function refreshColumns(input) {
        try {
            let inputCopy = input.slice();
            if (sortOrder === "Descending") inputCopy.reverse();
            const promises = inputCopy.map((status, index) => {
                let data = {
                    ...status
                };
                data[orderColumn] = index + 1;
                return API.saveRow(data).catch((error) => {
                    console.error(error);
                });
            });
            await Promise.all(promises);
            await fetchTables();
            notificationStore.actions.success(
                `Your list has been successfully rearranged.`
            );
            await fetchTables();
            console.log(labelColumn, orderColumn);
        } catch (error) {
        }
    }
    async function handleDrop(event) {
        if (dragIndex !== null && dropIndex !== null) {
            tableStatuses.splice(
                dropIndex,
                0,
                tableStatuses.splice(dragIndex, 1)[0]
            );
            refreshColumns(reactiveTableStatuses);
            //console.log(reactiveTableStatuses);
            reactiveTableStatuses = tableStatuses;
        }
        dragIndex = null;
        dropIndex = null;
    }

    onMount(() => {
        // define a Promise that resolves when the variable is populated
        function waitForVariable(dataProvider) {
            return new Promise(function (resolve) {
                var checkVariable = function () {
                    if (dataProvider) {
                        resolve();
                    } else {
                        setTimeout(checkVariable, 100); // check variable every 100ms
                    }
                };
                checkVariable();
            });
        }
        // wait for the variable to become populated, then run the process
        waitForVariable(dataProvider)
            .then(function () {
                fetchTables();
            })
            .catch(function (err) {
                console.log(err); // logging the error to the console.
            });
    });
</script>

<div on:drop={handleDrop}>
    <div class="spectrum-Table">
        {#if componentTitle.length !== 0 }
            <div class="spectrum-Table-head">
                <div class="spectrum-Table-headCell">
                    <div class="title">
                        {componentTitle}
                    </div>
                </div>
            </div>
        {/if}
        {#each reactiveTableStatuses as status, index}
            <div
                    class="spectrum-Table-row"
                    draggable="true"
                    on:dragstart={(event) => handleDragStart(event, index)}
                    on:dragover={(event) => handleDragOver(event, index)}
                    on:dragend={() => {
                        dragIndex = null;
                        dropIndex = null;
                    }}
            >
                <div class="spectrum-Table-cell">
                    {#if showOrder === true}{status[orderColumn]} - {/if}{status[labelColumn]}
                </div>
            </div>
        {/each}
    </div>
</div>