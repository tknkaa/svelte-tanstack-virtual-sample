<script lang="ts">
  export type CellType = {
    x: number;
    y: number;
    value: string;
  };

  interface Props {
    cell: CellType;
  }

  let { cell = $bindable() }: Props = $props();
</script>

<input bind:value={cell.value} />
