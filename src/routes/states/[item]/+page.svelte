<script>
    import { ItemStack } from "$lib/items";
    import { locale } from "$lib/stores";
    import { getTakeable, makeItemStack } from "$lib/utils";
    import Header from "../../Header.svelte";
    import ItemStackDetailDisplay from "../../ItemStackDetailDisplay.svelte";

    import { page } from "$app/stores";
    import { liveQuery } from "dexie";
    import Body from "../../Body.svelte";
    import SearchableItemList from "../../SearchableItemList.svelte";

    $: itemStack = liveQuery(async () => {
        return await makeItemStack(await getTakeable($page.params.item));
    });

    $: states = liveQuery(() => {
        return db.blockstates
            .where({ blockId: $page.params.item.split("[")[0] })
            .and((blockState) => blockState.fullId !== $page.params.item)
            .toArray();
    });

    const name = liveQuery(async () => {
        return await $itemStack?.getName($locale);
    });
</script>

<svelte:head>
    <title
        >Other blockstates of {$name || $itemStack?.fullId} - CR Recipes</title
    >
</svelte:head>

<Header />
<Body>
    <a href="/{window?.location?.search || ''}">Back to item list</a>
    <br /><br />

    <h2>Other blockstates of</h2>
    <ItemStackDetailDisplay
        itemStack={$itemStack || new ItemStack($page.params.item, 1)}
    />

    <div class="center">
        <SearchableItemList takeables={$states || []} />
        {#if !$states?.length}
            <p>{$name || $itemStack?.fullId} has no other blockstates.</p>
        {/if}
    </div>
</Body>

<style>
    .center {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
    }
</style>
