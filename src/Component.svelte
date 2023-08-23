<script>
    import { getContext, createEventDispatcher, onMount } from 'svelte';

    const dispatch = createEventDispatcher();

    const { API, notificationStore } = getContext('sdk');

    export let dataProvider;
    let dragIndex = null;
    let dropIndex = null;
    let tableStatuses = [];
    const statusTableId = dataProvider?.datasource.tableId;
    $: reactiveTableStatuses = tableStatuses;

    async function fetchTables() {
        try {
            const data = await API.fetchTableData(statusTableId);
            tableStatuses = data;
            if (tableStatuses.length > 0 && tableStatuses[0].hasOwnProperty('Order')) {
                tableStatuses.sort((a, b) => a.Order - b.Order);
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
            const promises = input.map((status, index) => {
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
            console.log(reactiveTableStatuses);
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
    <div class="component c94ae0a4dc5ed47428446c717c05e2f5a svelte-4sybt1 parent" data-id="c94ae0a4dc5ed47428446c717c05e2f5a" data-name="New Table" data-icon="Table" data-parent="cf31e6a91c9334617b4d18c77b97a0df9">
        <div class="spectrum--medium svelte-9e4ofs c94ae0a4dc5ed47428446c717c05e2f5a-dom" draggable="false" style="">
            <div class="wrapper svelte-rzhjll" style="--row-height: 55px; --header-height: 36px;">
                <div class="spectrum-Table svelte-rzhjll" style="grid-template-columns: minmax(auto, 1fr) minmax(auto, 1fr) minmax(auto, 1fr);">
                    <div class="spectrum-Table-head svelte-rzhjll">
                            <div class="spectrum-Table-headCell svelte-rzhjll">
                                <div class="title svelte-rzhjll">
                                    Title
                                </div>
                            </div>
                            <div class="spectrum-Table-headCell svelte-rzhjll">
                                <div class="title svelte-rzhjll">
                                    Order
                                </div>
                            </div>
                    </div>
                    {#each reactiveTableStatuses as status, index}
                        <div
                                class="spectrum-Table-row svelte-rzhjll"
                                draggable="true"
                                on:dragstart={(event) => handleDragStart(event, index)}
                                on:dragover={(event) => handleDragOver(event, index)}
                                on:dragend={() => {
                                    dragIndex = null;
                                    dropIndex = null;
                                }}
                        >
                            <div class="spectrum-Table-cell svelte-rzhjll">
                                <div class="svelte-16j2cvj" style="--max-cell-width: 200px;">
                                    <div class="svelte-1acrdjg">
                                        {status.Title}
                                    </div>
                                </div>
                            </div>
                            <div class="spectrum-Table-cell svelte-rzhjll">
                                <div class="svelte-16j2cvj" style="--max-cell-width: 200px;">
                                    <div class="svelte-1acrdjg">
                                        {status.Order}
                                    </div>
                                </div>
                            </div>
                        </div>
                    {/each}
                </div>
            </div>
        </div>
    </div>
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