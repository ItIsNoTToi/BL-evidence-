<script>
    import { ECharts } from '@evidence-dev/core-components';
    export let data = {
        total_records: 0,
        valid_records: 0,
        invalid_records: 0,
        valid_ratio_percent: 0
    };

    $: pie_data = [
        { value: data.valid_records, name: "Hợp lệ" },
        { value: data.invalid_records, name: "Không hợp lệ" }
    ];
</script>

<div style="margin: 10px 0%; font-weight: bold; font-size: large;">Bảng hiển thị</div>
<div class="container">
  <div class="content">
   <span>📦 Tổng số bản ghi: {data.total_records}</span><br>
    <span>✅ Số bản ghi hợp lệ: {data.valid_records}</span><br>
    <span>❌ Số bản ghi không hợp lệ: {data.invalid_records}</span><br>
    <span>📈 Tỷ lệ hợp lệ: {data.valid_ratio_percent}%</span>
  </div>
</div>

<!-- Biểu đồ -->
<div style="margin: 10px 0%; font-weight: bold; font-size: large;">Biểu đồ</div>
<ECharts
  config={{
    tooltip: { formatter: '{b}: {c} ({d}%)' },
    series: [{ type: 'pie', radius: '50%', data: pie_data }]
  }}
/>

<style>
  .container {
    padding: 1rem;
    border: 1px solid #ccc;
    border-radius: 8px;
    background-color: #f9f9f9;
  }
  .content {
    display: flex;
    flex-direction: column;
    font-size: 1.2rem;
    line-height: 1.5;
  }
</style>
