<script lang="ts">
  export let onClick: () => void;
  export let tall: boolean = false;
  export let value: string;
  export let id: string = undefined;
</script>

{#if tall}
  <input class="tall" type="button" on:click={onClick} {value} {id} />
{:else}
  <input type="button" on:click={onClick} {value} {id} />
{/if}

<style>
  input {
    border-radius: 0.2rem;
    padding: 2rem;
  }
  .tall {
    grid-row: span 2 / span 2;
  }
</style>
