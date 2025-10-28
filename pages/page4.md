---
title: "Báo cáo hoạt động User"
---

```sql summaryStatistics
  SELECT
    COUNT(*) AS Active_Users,
    COUNT(*) AS Total_Users,
    COUNT(*) FILTER (WHERE is_valid = TRUE) AS Published_User,
    COUNT(*) FILTER (WHERE is_valid = FALSE) AS Drafted_User,
    ROUND(
        (COUNT(*) FILTER (WHERE is_valid = TRUE)::numeric / COUNT(*) * 100),
        2
    ) AS valid_ratio_percent
  FROM mydatabase.t_giaythongbao
```

<BigValue 
  data={summaryStatistics} 
  value=Total_Users
  fmt=num1k
/>
<BigValue 
  data={summaryStatistics} 
  value=Active_Users
  fmt=num1k
/>
<BigValue 
  data={summaryStatistics} 
  value=Published_User
  title="Published User"
  comparison=Published_User
  fmt=num1k
  comparisonFmt=pct0
  comparisonTitle="User"
/>
<BigValue 
  data={summaryStatistics} 
  value=Drafted_User
  title="Drafted User"
  comparison=Published_User
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

<Chart data={DataBarchart} title="Phân tích trạng thái user theo giờ" >
    <Bar y=Drafted_User/>
    <Line y=Published_User/>
</Chart>

<BarChart 
  data={DataBarchart} 
  x=Detail_hour 
  y=Published_User 
  y2=Drafted_User 
  title="Phân tích trạng thái user theo giờ" 
/>

```sql Data
  SELECT
    jsonb_data -> 'ToChuc' ->  'HoatDongKinhDoanh' -> 'LoaiHinhKinhDoanh' -> 0 ->> 'username' AS "Tên người dùng",
    jsonb_data -> 'ToChuc' -> 'HoatDongKinhDoanh' -> 'GiayCapPhepKinhDoanh' ->> 'NgayCap' AS "Ngày truy cập",
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
```

<DataTable data={Data}/>