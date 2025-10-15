<script lang="ts">
    import { chart } from "svelte-apexcharts";
    export let dataQuery = []; // [{month: 'Jan', sales_usd: 30}, ...]

    // Reactive statement
    $: month = dataQuery.map(d => d.month);
    $: sales = dataQuery.map(d => d.sales_usd);

    $: options = {
        chart: { type: "bar" },
        series: [{ name: "Sales", data: sales }],
        xaxis: { categories: month }
    };
</script>

<div use:chart={options} />
