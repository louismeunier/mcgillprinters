<script lang="ts">
    let search;

    import printers from "./_building-printers.json";
    console.log(printers);
</script>

<div>
    <input type="text" bind:value="{search}"/>
    <ul>
       {#if search}
            {#each Object.keys(printers) as building}
                {#if building.toLowerCase().includes(search.toLowerCase())}
                    <li>{building}</li>
                    <!-- {#each printers[building] as printer}
                        <li>{printer.Room}</li>
                    {/each} -->
                {/if}
            {/each}
       {/if}
    </ul>
</div>

<style>
    ul {
        list-style: none;
    }
</style>