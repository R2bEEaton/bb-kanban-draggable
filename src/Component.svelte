<script>
    import {getContext, onMount} from 'svelte';
    import ProgressCircle from "./ProgressCircle/Popover.svelte"
    import CellRenderer from "./Table/CellRenderer.svelte"

    const { API, notificationStore } = getContext('sdk');

    export let componentTitle = "";
    export let dataProvider;
    export let labelColumn;
    export let orderColumn;
    export let sortOrder = "Descending";
    $: sortOrder, fetchTables();
    export let showOrder = true;
    export let loading = false;
    let dragIndex = null;
    let dropIndex = null;
    let tableStatuses = [];
    const statusTableId = dataProvider?.datasource.tableId;
    $: reactiveTableStatuses = tableStatuses;

    async function fetchTables() {
        try {
            tableStatuses = await API.fetchTableData(statusTableId);
            if (tableStatuses.length > 0) {
                tableStatuses.sort((a, b) => a[orderColumn] - b[orderColumn]);
            }
            if (sortOrder === "Descending") tableStatuses.reverse();
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
                console.log(err);
            });
    });

    const headerHeight = 36;
    const rowHeight = 46;
    $: totalRowCount = tableStatuses.length;
    $: rowCount = tableStatuses.length;
    $: visibleRowCount = tableStatuses.length;

    $: heightStyle = getHeightStyle(
        visibleRowCount,
        rowCount,
        totalRowCount,
        rowHeight,
        loading
    )

    $: gridStyle = getGridStyle(2);
    const getHeightStyle = (
        visibleRowCount,
        rowCount,
        totalRowCount,
        rowHeight,
        loading
    ) => {
        if (loading) {
            return `height: ${headerHeight + visibleRowCount * rowHeight}px;`
        }
        if (!rowCount || !visibleRowCount || totalRowCount <= rowCount) {
            return ""
        }
        return `height: ${headerHeight + visibleRowCount * rowHeight}px;`
    }

    const getGridStyle = (fields) => {
        let style = "grid-template-columns:"
        for (let i = 0; i < fields; i++) style += " minmax(auto, 1fr)";
        style += ";"
        return style
    }
</script>

<div class="wrapper" style={`--row-height: ${rowHeight}px; --header-height: ${headerHeight}px;`} on:drop={handleDrop}>
    {#if loading}
        <div class="loading" style={getHeightStyle()}>
            <slot name="loadingIndicator">
                <ProgressCircle />
            </slot>
        </div>
    {:else}
        <div class="spectrum-Table" class:no-scroll={!rowCount} style={`${heightStyle}${gridStyle}`}>
            {#if componentTitle.length !== 0 }
                <div class="spectrum-Table-head">
                    <div class="spectrum-Table-headCell">
                        <div class="title">{componentTitle}</div>
                    </div>
                </div>
            {/if}
            {#each reactiveTableStatuses as row, index}
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
                        <div class="spectrum-Table-cell">
                            <div>{row[labelColumn]}</div>
                        </div>
                        <!--{#if showOrder === true}{status[orderColumn]} - {/if}{status[labelColumn]}-->
                    </div>
                    <div class="spectrum-Table-cell">
                        <div class="spectrum-Table-cell">
                            <div>{row[orderColumn]}</div>
                        </div>
                        <!--{#if showOrder === true}{status[orderColumn]} - {/if}{status[labelColumn]}-->
                    </div>
                </div>
            {/each}
        </div>
    {/if}
</div>