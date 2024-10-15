<script>
    import { createEventDispatcher } from 'svelte';
    const dispatch = createEventDispatcher();
    export let columns;
    export let rows;
    export let sortBy = "";
    export let sortOrder = 1;
    export let classNameTable = '';
    export let classNameTbody = '';
    export let classNameRow = '';
    export let classNameCell = '';
    export let rowKeep = true;
    export let resetKeep = false;
    export let fillrow=[];
    export let fillcolor='floralwhite';

    let sortFunction = () => "";        
    let columnByKey = {};
    let rowbdcolor=[];
    let temprows;
    let rowbkcolor=[];
    columns.forEach(col => {
        columnByKey[col.key] = col;
    });
    $: c_rows = rows            
            .map(r => (Object.assign({}, r, {$sortOn: sortFunction(r)} ) ) )
            .sort((a, b) => {
                if (a.$sortOn > b.$sortOn) return sortOrder;
                else if (a.$sortOn < b.$sortOn) return -sortOrder;
                return 0;
            });

    const asStringArray = (v) => [].concat(v).filter(v => typeof v === 'string' && v !== "").join(' ');
    
    $: {
        let col = columnByKey[sortBy];
        if (col !== undefined && col.sortable === true && typeof col.value === "function") {
            sortFunction = r => col.value(r);
        }
    };
    $: {        
        if (rows[0] !== undefined){
            if (temprows!==rows  | resetKeep )  {
                rowbdcolor.forEach((value, index) => {
                    rowbdcolor[index] = "";
                });
                resetKeep=false;
            }
            temprows=rows;
        }
        if (fillrow.length!==0){
            rowbkcolor.fill("");
            fillrow.forEach( item => {
                rowbkcolor[item]=fillcolor;
            })
        } else{
            rowbkcolor=rowbkcolor.fill("");

        }
    };
    
    const handleClickRow = (event, row, No) => {
        dispatch('clickRow', {event, row} );
        if (rowKeep=== true) {            
            rowbdcolor.fill("");
            rowbdcolor[No] = "solid red ";           
        }
    }

    const handleClickCell = (event, row, key) => {
        dispatch('clickCell', {event, row, key} );
    }
</script>

<style>
    table {
        width: 100%;
        font-size: inherit;
        /*background: #6fb1f8;*/
    }
   
    tr th select {
        width: 100%;
    }
    /*tbody td:hover  {*/
        /*background-color:lightgoldenrodyellow;*/
    /*    color:black;*/
    /*}*/

    tr td{
        display: block;
        /*margin-top: 1px;*/
        /*margin-bottom: 1px;*/
        border-radius: 6px;
        padding-left: 6px;
        padding-right: 2px;        
    }

</style>

<table class={asStringArray(classNameTable)}>
    <tbody class={asStringArray(classNameTbody)}>
    {#each c_rows as row, n}
        <slot name="row" row={row} n={n} >
            <tr  on:click={(e)=>{handleClickRow(e, row,n)}} class={asStringArray(classNameRow)}>
                {#each columns as col, index}
                   {#if col.key==="id"}
                     <td style="display:none" ></td>
                   {:else}
                    <td style="border: {rowbdcolor[n]}; background: {rowbkcolor[n]} "
                            on:click={(e)=>{handleClickCell(e, row, col.key)}}
                            class={asStringArray([col.class, classNameCell])}
                    >{@html col.renderValue ? col.renderValue(row) : col.value(row)}</td>
                   {/if}
                {/each}
            </tr>
        </slot>
    {/each}
    </tbody>
</table>