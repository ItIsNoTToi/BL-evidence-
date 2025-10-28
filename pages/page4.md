---
title: "Báo cáo hoạt động User"
---

```sql summaryStatistics
  SELECT
    COUNT(*) FILTER (WHERE is_valid = TRUE) AS Active_Users,
    COUNT(*) AS Total_Users,
    COUNT(*) FILTER (WHERE is_valid = false) AS Unactive_Users,
    COUNT(*) FILTER (WHERE is_valid = TRUE) AS Published_User,
    COUNT(*) FILTER (WHERE is_valid = FALSE) AS Drafted_User,
    COUNT(*) FILTER (WHERE is_valid = FALSE) AS DraftedOver30Days_User
  FROM mydatabase.t_giaythongbao
```
Thống kê tổng quan<Info description="Thông tin thống kê tổng quan" />
<BigValue 
  data={summaryStatistics} 
  value=Total_Users
  fmt=num1k
  title="👥 Tổng số user"
/>
<BigValue 
  data={summaryStatistics} 
  value=Active_Users
  fmt=num1k
  comparison=Total_Users
  title="🟢 User đang hoạt động (Active)"
  comparisonFmt=pct0
  comparisonTitle="User"
/>
<BigValue 
  data={summaryStatistics} 
  value=Unactive_Users
  fmt=num1k
  comparison=Total_Users
  title="🔴 User bị cấm (Banned)"
  comparisonFmt=pct0
  comparisonTitle="User"
/>
<BigValue 
  data={summaryStatistics} 
  value=Published_User
  title="🕓 User đăng nhập hôm nay"
  comparison=Total_Users
  fmt=num1k
  comparisonFmt=pct0
  comparisonTitle="User"
/>
<BigValue 
  data={summaryStatistics} 
  value=Drafted_User
  title="🔐 User đăng xuất hôm nay"
  comparison=Total_Users
  fmt=num1k
  comparisonFmt=pct0
  comparisonTitle="User"
/>
<BigValue 
  data={summaryStatistics} 
  value=DraftedOver30Days_User
  title="⚠️ User chưa hoạt động > 30 ngày"
  fmt=num1k
  comparisonFmt=pct0
  comparisonTitle="User"
/>

```sql getdata
  select
    (jsonb_data -> 'CoQuanCap' ->> 'TenCoSoToChuc') as TenCoSoToChuc,
  from mydatabase.t_giaythongbao
  GROUP BY TenCoSoToChuc
```

<Dropdown 
  data={getdata} 
  name=So_value
  value=TenCoSoToChuc 
  title="Chọn sở:"
  defaultValue="Sở Xây dựng tỉnh Bắc Ninh"
/>

```sql date
  SELECT created_at AS day
  FROM mydatabase.t_giaythongbao
  ORDER BY day DESC;
```

<DateInput
  name=pick_date 
  title="Chọn ngày:"
  data={date}
  dates=day
/>

```sql DataBarchart
  SELECT
    EXTRACT(HOUR FROM created_at) AS Detail_hour,
    COUNT(*) FILTER (WHERE is_valid = TRUE) AS Published_User,
    COUNT(*) FILTER (WHERE is_valid = FALSE) AS Drafted_User
  FROM mydatabase.t_giaythongbao
  WHERE 
    (jsonb_data -> 'CoQuanCap' ->> 'TenCoSoToChuc') = '${inputs.So_value.value}'
    AND created_at::timestamp >= '${inputs.pick_date.value}'
  GROUP BY Detail_hour
  ORDER BY Detail_hour;
```

Phân tích trạng thái login/logout tài khoản user <Info description="Phân tích trạng thái login/logout tài khoản của user theo sở" />

<Grid cols=2 >
  <Chart data={DataBarchart} title="BarChart + Line Chart" >
      <Bar y=Drafted_User/>
      <Line y=Published_User/>
  </Chart>
  <BarChart 
    data={DataBarchart} 
    x=Detail_hour 
    y=Drafted_User
    y2=Published_User  
    title="BarChart" 
  />
</Grid>

<TextInput
  name=username
  title="Tìm theo Username:"
  defaultValue="%"
/>
<Checkbox
    title="Top 5" 
    name=gettop5
/>
<Info description="5 tài khoản có lượng truy cập cao nhất" />

```sql Data
  SELECT
    jsonb_data -> 'ToChuc' ->  'HoatDongKinhDoanh' -> 'LoaiHinhKinhDoanh' -> 0 ->> 'username' AS "Tên người dùng",
    jsonb_data -> 'ToChuc' -> 'HoatDongKinhDoanh' -> 'GiayCapPhepKinhDoanh' ->> 'NgayCap' AS "Thời gian hoạt động",
    created_at::timestamp as "Ngày truy cập", 
    CASE 
      WHEN is_valid = TRUE THEN 'Đã duyệt'
      WHEN is_valid = FALSE THEN 'Nháp'
      ELSE 'Không xác định'
    END AS "Trạng thái"
  FROM mydatabase.t_giaythongbao
  WHERE 
    (jsonb_data -> 'CoQuanCap' ->> 'TenCoSoToChuc') = '${inputs.So_value.value}'
    AND created_at::timestamp >= '${inputs.pick_date.value}' 
    AND (jsonb_data -> 'ToChuc' ->  'HoatDongKinhDoanh' -> 'LoaiHinhKinhDoanh' -> 0 ->> 'username') like '${inputs.username}'
  ORDER BY COUNT(*) OVER (PARTITION BY (jsonb_data -> 'ToChuc' -> 'HoatDongKinhDoanh' -> 'LoaiHinhKinhDoanh' -> 0 ->> 'username')) DESC
  LIMIT CASE WHEN ${inputs.gettop5} THEN 5 ELSE NULL END
```

<DataTable 
data={Data}
title="Bảng thông tin tài khoản hoạt động theo Sở"
rows=10
search=true
/>

Danh sách tài khoản bị cấm / đang chờ xử lý <Info description="Danh sách thông tin tài khoản bị cấm hoặc đang chờ xử lý" />

<TextInput
  name=username_ban
  title="Tìm theo Username:"
  defaultValue="%"
/>

```sql DataUserban
  SELECT
    jsonb_data -> 'ToChuc' ->  'HoatDongKinhDoanh' -> 'LoaiHinhKinhDoanh' -> 0 ->> 'username' AS "Tên người dùng",
    jsonb_data -> 'ToChuc' -> 'HoatDongKinhDoanh' -> 'GiayCapPhepKinhDoanh' ->> 'NgayCap' AS "Thời gian hoạt động",
    created_at::timestamp as "Ngày truy cập"
  FROM mydatabase.t_giaythongbao
  WHERE 
    (jsonb_data -> 'CoQuanCap' ->> 'TenCoSoToChuc') = '${inputs.So_value.value}'
    AND created_at::timestamp >= '${inputs.pick_date.value}' 
    AND (jsonb_data -> 'ToChuc' ->  'HoatDongKinhDoanh' -> 'LoaiHinhKinhDoanh' -> 0 ->> 'username') like '${inputs.username_ban}'
    AND is_valid = false
```

<DataTable 
data={DataUserban}
title="Bảng thông tin tài khoản bị cấm / đang chờ xử lý"
rows=10
search=true
/>

Phân bố user theo trạng thái

```sql pie_query
  SELECT
    COUNT(*) FILTER (WHERE is_valid = TRUE)  AS Published_User,
    COUNT(*) FILTER (WHERE is_valid = FALSE) AS Drafted_User,
    COUNT(*)                                 AS Total_Users
  FROM mydatabase.t_giaythongbao
  WHERE 
    (jsonb_data -> 'CoQuanCap' ->> 'TenCoSoToChuc') = '${inputs.So_value.value}'
    AND created_at::timestamp >= '${inputs.pick_date.value}';
```

```sql pie_data
SELECT 'Đã duyệt' AS name, Published_User AS value FROM ${pie_query}
UNION ALL
SELECT 'Nháp' AS name, Drafted_User AS value FROM ${pie_query}
UNION ALL
SELECT 'Hoạt động' AS name, (Published_User + Drafted_User) AS value FROM ${pie_query}
UNION ALL
SELECT 'Không hoạt động' AS name, (Total_Users - (Published_User + Drafted_User)) AS value FROM ${pie_query};
```

<ECharts config={{
  tooltip: {
    trigger: 'item',
    formatter: '{b}: {c} ({d}%)'
  },
  legend: {
    orient: 'vertical',
    left: 'left'
  },
  series: [
    {
      type: 'pie',
      radius: '60%',
      data: [...pie_data],
      emphasis: {
        itemStyle: {
          shadowBlur: 10,
          shadowOffsetX: 0,
          shadowColor: 'rgba(0, 0, 0, 0.5)'
        }
      }
    }
  ]
}} />

