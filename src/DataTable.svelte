<script>
    import {onMount} from "svelte"

    export let columns = [];
    export let rows = [];
    export let manager = {
        updates: [],
        callbacks: {
            update: [],
            sort: []
        },
        sortIndex: 0,
        sortDirection: 'asc'
    };

    $: hasCheckbox = columns.length > 0 && columns.map(n => n.id).indexOf('checkbox') >= 0;
    $: dataIsArray = rows.length > 0 && Array.isArray(rows[0]);

    manager.sort = (columnIndex, direction) => {
        let initColIndex = columnIndex;

        if(!dataIsArray) {
            columnIndex = columns[columnIndex]['id'];
        }

        rows.sort((a, b) => {
            let ascValue = (direction === 'asc' || direction === 'ascending') ? -1 : 1;
            let aVal = a[columnIndex] !== undefined ? a[columnIndex] : 0;
            let bVal = b[columnIndex] !== undefined ? b[columnIndex] : 0;

            if(aVal === bVal)
                return 0;

            return (aVal <= bVal ? (1 * ascValue) : (-1 * ascValue));
        });

        manager.sortIndex = initColIndex;
        rows = rows;
    };

    manager.sortBy = (colIndex) => {
        if(colIndex === manager.sortIndex)
            manager.sortDirection = manager.sortDirection === 'asc' ? 'desc' : 'asc';
        else
            manager.sortDirection = 'asc';

        manager.sort(colIndex, manager.sortDirection);
    };
    
    manager.on = (event, callback) => {
        manager.callbacks[event].push(callback);
    };

    onMount(async() => {
        // TODO: Stub
    });

    let fireUpdateEvent = (rowIndex, colIndex, newValue) => {
        rows[rowIndex][colIndex] = newValue;
        for(let callback of manager.callbacks['update']) {
            callback(rowIndex, colIndex, newValue);
        }
    }

    let handleRowSelection = (rowIndex, colIndex, newValue) => {
        // let selectedItem = {rowIndex, colIndex};
        // let selectedIndex = manager.selected.indexOf(selectedItem);

        // if(newValue === false && selectedIndex > -1) {
        //     manager.selected.splice(selectedIndex);
        // } else if(newValue === true && selectedIndex === -1) {
        //     manager.selected.push(selectedItem)
        // }
        
        if(dataIsArray) {
            rows[rowIndex][colIndex] = newValue;
        } else {
            rows[rowIndex][columns[colIndex].id] = newValue;
        }

        fireUpdateEvent(rowIndex, colIndex, newValue);
    }

    let handleSort = (event, colIndex) => {
        manager.sortBy(colIndex);

        for(let callback of manager.callbacks['sort']) {
            callback(colIndex, columns[colIndex]);
        }
    }
</script>

<style type="text/css">
    table{font-size:12px;color:#333333;width:100%;border-width: 1px;border-color: #729ea5;border-collapse: collapse;}
    th {user-select:none;cursor:pointer;font-size:12px;background-color:#acc8cc;border-width: 1px;padding: 1px;border-style: solid;border-color: #729ea5;text-align:center;}
    tr {background-color:#ffffff;}
    td {font-size:12px;border-width: 1px;padding: 1px;border-style: solid;border-color: #729ea5; text-align: left}
    .editable-cell {border: none; height: 100%; width: 100%; margin: 0}
    .checkbox-cell {text-align: center;}
    tr:hover {background-color:#ffff99;}
</style>

<div id="datable-container">

    <table id="datatable-table">
        <thead>
        {#each columns as column, colIndex}
            {#if column['visible'] !== false}
                {#if column['sortable'] === true}
                <th on:click={(event) => handleSort(event, colIndex)}>
                    <div class="datatable-header-label">{column.label} 
                        {#if manager.sortIndex === colIndex}
                            <span>{manager.sortDirection === 'asc' ? '↓' : '↑'}</span>
                        {/if}
                    </div>
                </th>
                {:else}
                <th>
                    <div class="datatable-header-label">{column.label}</div>
                </th>
                {/if}
            {/if}
        {/each}
        </thead>
        <tbody>
            <!--If the data array is a simple 2 dimensional array-->
            {#if rows.length > 0 && Array.isArray(rows[0])}
                {#each rows as row, rowIndex}
                <tr>
                    {#each row as col, i}
                        {#if columns[i]['visible'] !== false}
                            {#if columns[i]['type'] !== 'checkbox' && columns[i]['editable'] === true }
                                <td>
                                    <input class="editable-cell" type="text" value={col} on:input={(event) => fireUpdateEvent(rowIndex, i, event.target.value)}>
                                </td>
                            {:else if columns[i]['type'] === 'checkbox'}
                                <td class="checkbox-cell">
                                    <input  checked={col} type="checkbox" id={'checkbox' + i} on:change={(event) => handleRowSelection(rowIndex, i, event.target.checked)}>
                                </td>
                            {:else}
                                <td>{col}</td>
                            {/if}
                        {/if}
                    {/each}
                </tr>
                {/each}
            {:else}

            <!-- If the data array is an array of objects -->
                {#each rows as row, rowIndex}
                <tr>
                    <!-- Loop through predefined columns -->
                    {#each columns as col, i}
                        {#if col['visible'] !== false}

                            {#if col['type'] !== 'checkbox' && col['editable'] === true }
                                <td>
                                    <input class="editable-cell" type="text" value={row[col.id]} on:input={(event) => fireUpdateEvent(rowIndex, i, event.target.value)}>
                                </td>
                            {:else if col['type'] === 'checkbox'}
                                <td class="checkbox-cell">
                                    <input  checked={row[col.id]} type="checkbox" id={'checkbox' + i} on:change={(event) => handleRowSelection(rowIndex, i, event.target.checked)}>
                                </td>
                            {:else}
                                <td>{row[col.id]}</td>
                            {/if}


                        {/if}
                    {/each}
                </tr>
                {/each}
            {/if}
        </tbody>
    </table>
</div>