# Bb-list-draggable
This is the read me for the draggable/sortable list plugin.

# Description
With this plugin you are able to order a list by dragging the different elements around in the browser.

## Features
* Reorder a list by dragging
* Higher priority on top

## Instructions
* Create a table with the following fields.
    * **Title**
    * **Order**
    * You can add as many other fields as you want, but the first two are required because they work directly with the functionality of the plugin.

    ### Additional Notes
    * Rows with a higher Order will be at the top of the list.
    * Make sure that Order is a unique integer as the sorting depends on it.

## Demo
![list-draggable-demo](https://github.com/R2bEEaton/bb-kanban-draggable/assets/34921506/54985b26-7b91-4b18-8f24-4f8fe4e38292)

## Disclaimer
* Most of this plugin was copied/modified from the column reordering capability of @ConorWebb96's [Kanban plugin](https://github.com/ConorWebb96/bb-kanban-draggable/blob/main/src/components/ColumnsSort.svelte)
* I initially wanted to make the sorting work directly in a table, but I feel that this is better. You can display the sorted data with a table and have a "reorder" button that opens a sidebar with the Draggable List element. Then, use an [Interval](https://github.com/MartinPicc/budibase-interval-plugin) (one of my favorite plugins) to refresh a dataprovider on the table periodically. If you use an external database, you can get away with refreshing pretty often.
