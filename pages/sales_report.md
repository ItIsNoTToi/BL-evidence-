---
title: "Báo cáo doanh thu"
inputs:
  category:
    title: "Danh mục"
    default: "%"
  month:
    title: "Tháng"
    default: "%"
---

<script>
    import CategoryFilter from '../../../../../src/lib/ui/CategoryFilter.svelte';
    import YearFilter from '../../../../../src/lib/ui/YearFilter.svelte';
    import InteractiveSalesChart from '../../../../../src/lib/ui/InteractiveSalesChart.svelte';
</script>

```sql categories
    select category
    from needful_things.orders
    group by category
```

<CategoryFilter categories={categories} inputs={inputs} onchange={(val) => inputs.category.value = val} />
<YearFilter inputs={inputs} onchange={(val) => inputs.year.value = val}/>

```sql sales_data
    select 
        date_trunc('month', order_datetime) as month,
        median(sales) as sales_usd,
        category
    from needful_things.orders
    where category like '${inputs.category.value}' || '%'
    and date_part('year', order_datetime)::text like '${inputs.year.value}' || '%'
    group by month, category
    order by sales_usd desc
```
<InteractiveSalesChart dataQuery={sales_data}/>