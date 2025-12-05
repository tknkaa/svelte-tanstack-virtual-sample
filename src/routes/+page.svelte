<script lang="ts">
  import { createVirtualizer } from "@tanstack/svelte-virtual";
  import type { Readable } from "svelte/store";
  import type { SvelteVirtualizer } from "@tanstack/svelte-virtual";
  import Cell from "$lib/Cell.svelte";
  import type { CellType } from "$lib/Cell.svelte";

  let grid: CellType[][] = $state([]);
  function getCell(x: number, y: number): CellType {
    if (!grid[y]) {
      grid[y] = [];
    }
    let cell = grid[y][x];
    if (!cell) {
      cell = {
        x,
        y,
        value: "",
      };
      grid[y][x] = cell;
    }
    return cell;
  }
  function setCell(cell: CellType) {
    grid[cell.y][cell.x] = cell;
  }

  let virtualListEl: HTMLDivElement;

  let rowVirtualizer:
    | Readable<SvelteVirtualizer<HTMLDivElement, HTMLDivElement>>
    | undefined = $state();

  let columnVirtualizer:
    | Readable<SvelteVirtualizer<HTMLDivElement, HTMLDivElement>>
    | undefined = $state();

  $effect(() => {
    rowVirtualizer = createVirtualizer<HTMLDivElement, HTMLDivElement>({
      count: 10000,
      getScrollElement: () => virtualListEl,
      estimateSize: () => 35,
      overscan: 5,
    });
    columnVirtualizer = createVirtualizer<HTMLDivElement, HTMLDivElement>({
      horizontal: true,
      count: 10000,
      getScrollElement: () => virtualListEl,
      estimateSize: () => 100,
      overscan: 5,
    });
  });
</script>

<div class="list scroll-container" bind:this={virtualListEl}>
  <div
    style="position: relative; height: {$rowVirtualizer?.getTotalSize()}px; width: {$columnVirtualizer?.getTotalSize()}px;"
  >
    {#each $rowVirtualizer?.getVirtualItems() as row (row.index)}
      {#each $columnVirtualizer?.getVirtualItems() as col (col.index)}
        <div
          class={col.index % 2
            ? row.index % 2 === 0
              ? "list-item-odd"
              : "list-item-even"
            : row.index % 2
              ? "list-item-odd"
              : "list-item-even"}
          style="position: absolute; top: 0; left: 0; width: {col.size}px; height: {row.size}px; transform: translateX({col.start}px) translateY({row.start}px);"
        >
          <Cell
            bind:cell={
              () => getCell(col.index, row.index), (cell) => setCell(cell)
            }
          />
        </div>
      {/each}
    {/each}
  </div>
</div>

<style>
  .scroll-container {
    height: 500px;
    width: 500px;
    overflow: auto;
  }
</style>
