<script context="module">
  export const evidenceInclude = true;
</script>

<script>
  import { onMount } from "svelte"; 
  export let inputs;
  export let onchange = () => {};
  onMount(() => {
    if (!inputs.year?.value) inputs.year.value = "%";
  });
</script>

<div>
  <span>Năm:</span>
  <select bind:value={inputs.year.value} on:change={() => onchange(inputs.year.value)}>
    <option value="%" selected>Tất cả</option>
    {#each [2019,2020,2021] as y}
      <option value={y}>{y}</option>
    {/each}
  </select>
</div>
