---
title: "Báo cáo tổng hợp trạng thái dữ liệu"
description: "Báo cáo theo dõi chất lượng và trạng thái dữ liệu trong hệ thống."
---

```sql summaryStatistics
    SELECT
        COUNT(*) AS total_records,
        COUNT(*) FILTER (WHERE is_valid = TRUE) AS valid_records,
        COUNT(*) FILTER (WHERE is_valid = FALSE) AS invalid_records,
        ROUND(
            (COUNT(*) FILTER (WHERE is_valid = TRUE)::numeric / COUNT(*) * 100),
            2
        ) AS valid_ratio_percent
    FROM mydatabase.t_giaythongbao;
```

<ShowBL 
 data={summaryStatistics[0]} 
/>