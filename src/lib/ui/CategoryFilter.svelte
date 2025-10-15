<script>
  import { onMount } from "svelte";
  export let categories = [];
  export let inputs; // truyền toàn bộ object inputs từ .md
  export let onchange = () => {};
  onMount(() => {
    if (!inputs.category.value) inputs.category.value = '%';
  });
</script>

<div>
  <span>Danh mục:</span>
  <select bind:value={inputs.category.value} on:change={() => onchange(inputs.category.value)}>
    <option value="%" selected>Tất cả</option>
    {#each categories as c}
      <option value={c.category}>{c.category}</option>
    {/each}
  </select>
</div>
