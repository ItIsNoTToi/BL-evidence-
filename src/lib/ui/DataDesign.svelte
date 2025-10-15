<script lang="ts">
  export let data = [];

  // Hàm định dạng tiền tệ USD
  const formatCurrency = (value) => {
    return new Intl.NumberFormat("en-US", {
      style: "currency",
      currency: "USD",
      maximumFractionDigits: 0,
    }).format(value || 0);
  };
</script>

<div class="table-container">
  <h2 class="title">💹 Sales by Category and Month</h2>
  <table class="styled-table">
    <thead>
      <tr>
        <th>#</th>
        <th>Month</th>
        <th>Category</th>
        <th>Sales (USD)</th>
      </tr>
    </thead>
    <tbody>
      {#each data as row, index}
        <tr>
          <td>{index + 1}</td>
          <td>{new Date(row.month).toLocaleDateString('en-US', { month: 'short', year: 'numeric' })}</td>
          <td>{row.category}</td>
          <td class="value-cell">{formatCurrency(row.sales_usd)}</td>
        </tr>
      {/each}
    </tbody>
  </table>
</div>

<style>
  .table-container {
    margin: 2rem auto;
    padding: 1.5rem;
    border-radius: 16px;
    background: linear-gradient(145deg, #f8fafc, #eef2f3);
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.08);
    max-width: 850px;
    text-align: center;
    transition: all 0.3s ease;
  }

  .title {
    color: #2563eb;
    font-size: 1.4rem;
    font-weight: 700;
    margin-bottom: 1.2rem;
  }

  .styled-table {
    width: 100%;
    border-collapse: collapse;
    background-color: white;
    border-radius: 10px;
    overflow: hidden;
  }

  thead {
    background-color: #2563eb;
    color: white;
  }

  th, td {
    padding: 0.9rem;
    font-size: 0.95rem;
  }

  tbody tr:nth-child(even) {
    background-color: #f9fafb;
  }

  tbody tr:hover {
    background-color: #e0f2fe;
    transition: 0.2s;
  }

  .value-cell {
    font-weight: 600;
    color: #047857;
  }

  th:first-child, td:first-child {
    width: 50px;
  }
</style>
