---
title: "Báo cáo tổng hợp trạng thái dữ liệu"
description: "Báo cáo theo dõi chất lượng và trạng thái dữ liệu trong hệ thống."

---

```sql total_orders
    SELECT
        COUNT(*) AS total_orders
    FROM mydatabase.orders;
```

<ShowTable total={total_orders} />