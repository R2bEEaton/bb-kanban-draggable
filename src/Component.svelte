<script>
    import { getContext, createEventDispatcher, onMount } from 'svelte';

    const dispatch = createEventDispatcher();

    const { API, notificationStore } = getContext('sdk');

    export let table;
    let dragIndex = null;
    let dropIndex = null;
    let tableStatuses = [];
    const statusTableId = table?.tableId;
    $: reactiveTableStatuses = tableStatuses;

    async function fetchTables() {
        try {
            const data = await API.fetchTableData(statusTableId);
            tableStatuses = data;
            if (tableStatuses.length > 0 && tableStatuses[0].hasOwnProperty('Order')) {
                tableStatuses.sort((a, b) => a.Order - b.Order);
            }
            console.log(tableStatuses);
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
            const promises = input.map((status, index) => {
                console.log(status, index);
                return API.saveRow({
                    ...status,
                    Order: index + 1,
                }).catch((error) => {
                    console.error(error);
                });
            });
            await Promise.all(promises);
            await fetchTables();
            notificationStore.actions.success(
                `Your column orders have been successfully rearranged.`
            );
            await fetchTables();
        } catch (error) {
            console.log(error);
        }
    }
    async function handleDrop(event) {
        if (dragIndex !== null && dropIndex !== null) {
            tableStatuses.splice(
                dropIndex,
                0,
                tableStatuses.splice(dragIndex, 1)[0]
            );
            refreshColumns(tableStatuses);
            reactiveTableStatuses = tableStatuses;
        }
        dragIndex = null;
        dropIndex = null;
    }

    onMount(() => {
        // define a Promise that resolves when the variable is populated
        function waitForVariable(table) {
            return new Promise(function (resolve) {
                var checkVariable = function () {
                    if (table) {
                        resolve();
                    } else {
                        setTimeout(checkVariable, 100); // check variable every 100ms
                    }
                };
                checkVariable();
            });
        }
        // wait for the variable to become populated, then run the process
        waitForVariable(table)
            .then(function () {
                fetchTables();
            })
            .catch(function (err) {
                console.log(err); // logging the error to the console.
            });
    });
</script>

<div on:drop={handleDrop}>
    {#each reactiveTableStatuses as status, index}
        <div
        class="status-item spectrum-Card-content"
        draggable="true"
        on:dragstart={(event) => handleDragStart(event, index)}
        on:dragover={(event) => handleDragOver(event, index)}
        on:dragend={() => {
            dragIndex = null;
            dropIndex = null;
        }}
        >
            {status.Title}
        </div>
    {/each}
</div>

<style>
    .status-item {
        display: block!important;
        height: auto;
        padding: 10px;
        margin-bottom: 10px;
        background-color: var(--spectrum-textfield-m-background-color, var(--spectrum-global-color-gray-50));
        border-color: var(--spectrum-textfield-m-border-color, var(--spectrum-alias-border-color));
        color: var(--spectrum-textfield-m-text-color, var(--spectrum-alias-text-color));
        cursor: pointer;
    }
</style>