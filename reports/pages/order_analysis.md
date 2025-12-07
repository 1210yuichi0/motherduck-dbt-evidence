---
sources:
  - motherduck_test
---
# 📊 注文分析ダッシュボード

## フィルター

```sql categories
select distinct category
from orders
order by category
```

```sql years
select distinct date_part('year', order_datetime) as year
from orders
order by year desc
```

<Dropdown data={categories} name=category value=category>
    <DropdownOption value="%" valueLabel="すべてのカテゴリ"/>
</Dropdown>

<Dropdown data={years} name=year value=year>
    <DropdownOption value="%" valueLabel="すべての年"/>
</Dropdown>

<ButtonGroup name=sales_range>
    <ButtonGroupItem valueLabel="すべて" value="%" default/>
    <ButtonGroupItem valueLabel="$50未満" value="low"/>
    <ButtonGroupItem valueLabel="$50-$200" value="medium"/>
    <ButtonGroupItem valueLabel="$200以上" value="high"/>
</ButtonGroup>

---

```sql order_summary
select
    count(distinct id) as total_orders,
    round(avg(sales), 2) as avg_order_value,
    round(sum(sales), 2) as total_revenue
from orders
where category like '${inputs.category.value}'
and date_part('year', order_datetime)::text like '${inputs.year.value}'
and case
    when '${inputs.sales_range}' = 'low' then sales < 50
    when '${inputs.sales_range}' = 'medium' then sales between 50 and 200
    when '${inputs.sales_range}' = 'high' then sales > 200
    else true
end
```

## 主要指標

<BigValue
    data={order_summary}
    value=total_orders
    title="総注文数"
/>

<BigValue
    data={order_summary}
    value=avg_order_value
    title="平均注文額"
    fmt=usd2
/>

<BigValue
    data={order_summary}
    value=total_revenue
    title="総収益"
    fmt=usd2
/>

## カテゴリ別売上

```sql category_sales
select
    category,
    count(*) as order_count,
    round(sum(sales), 2) as total_sales
from orders
where date_part('year', order_datetime)::text like '${inputs.year.value}'
and case
    when '${inputs.sales_range}' = 'low' then sales < 50
    when '${inputs.sales_range}' = 'medium' then sales between 50 and 200
    when '${inputs.sales_range}' = 'high' then sales > 200
    else true
end
group by category
order by total_sales desc
```

<BarChart
    data={category_sales}
    x=category
    y=total_sales
    title="カテゴリ別売上 ({inputs.year.label})"
    swapXY=true
/>

<DataTable data={category_sales} />

## 月別トレンド

```sql monthly_trend
select
    date_trunc('month', order_datetime) as month,
    count(*) as order_count,
    round(sum(sales), 2) as revenue
from orders
where category like '${inputs.category.value}'
and date_part('year', order_datetime)::text like '${inputs.year.value}'
group by 1
order by 1
```

<LineChart
    data={monthly_trend}
    x=month
    y=revenue
    title="月別売上推移 - {inputs.category.label}"
/>

<LineChart
    data={monthly_trend}
    x=month
    y=order_count
    title="月別注文数 - {inputs.category.label}"
/>

## 最近の注文 (TOP 50)

```sql recent_orders
select
    id,
    order_datetime,
    category,
    round(sales, 2) as sales
from orders
where category like '${inputs.category.value}'
and date_part('year', order_datetime)::text like '${inputs.year.value}'
and case
    when '${inputs.sales_range}' = 'low' then sales < 50
    when '${inputs.sales_range}' = 'medium' then sales between 50 and 200
    when '${inputs.sales_range}' = 'high' then sales > 200
    else true
end
order by order_datetime desc
limit 50
```

<DataTable data={recent_orders} />

## 注文額別分布

```sql sales_distribution
select
    case
        when sales < 20 then '< $20'
        when sales < 50 then '$20 - $50'
        when sales < 100 then '$50 - $100'
        when sales < 200 then '$100 - $200'
        else '$200+'
    end as value_bucket,
    count(*) as order_count
from orders
where category like '${inputs.category.value}'
and date_part('year', order_datetime)::text like '${inputs.year.value}'
group by 1
order by 1
```

<BarChart
    data={sales_distribution}
    x=value_bucket
    y=order_count
    title="注文額帯別分布"
/>
