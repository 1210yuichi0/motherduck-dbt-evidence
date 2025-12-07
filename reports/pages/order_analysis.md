---
sources:
  - motherduck_test
---
# 📊 注文分析ダッシュボード

## フィルター

```sql statuses
select distinct status
from orders
order by status
```

```sql years
select distinct date_part('year', order_date) as year
from orders
order by year desc
```

<Dropdown data={statuses} name=status value=status>
    <DropdownOption value="%" valueLabel="すべてのステータス"/>
</Dropdown>

<Dropdown data={years} name=year value=year>
    <DropdownOption value="%" valueLabel="すべての年"/>
</Dropdown>

<ButtonGroup name=amount_range>
    <ButtonGroupItem valueLabel="すべて" value="%" default/>
    <ButtonGroupItem valueLabel="$50未満" value="low"/>
    <ButtonGroupItem valueLabel="$50-$200" value="medium"/>
    <ButtonGroupItem valueLabel="$200以上" value="high"/>
</ButtonGroup>

---

```sql order_summary
select
    count(distinct order_id) as total_orders,
    round(avg(amount), 2) as avg_order_value,
    round(sum(amount), 2) as total_revenue
from orders
where status like '${inputs.status.value}'
and date_part('year', order_date)::text like '${inputs.year.value}'
and case
    when '${inputs.amount_range}' = 'low' then amount < 50
    when '${inputs.amount_range}' = 'medium' then amount between 50 and 200
    when '${inputs.amount_range}' = 'high' then amount > 200
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

## ステータス別売上

```sql status_sales
select
    status,
    count(*) as order_count,
    round(sum(amount), 2) as total_sales
from orders
where date_part('year', order_date)::text like '${inputs.year.value}'
and case
    when '${inputs.amount_range}' = 'low' then amount < 50
    when '${inputs.amount_range}' = 'medium' then amount between 50 and 200
    when '${inputs.amount_range}' = 'high' then amount > 200
    else true
end
group by status
order by total_sales desc
```

<BarChart
    data={status_sales}
    x=status
    y=total_sales
    title="ステータス別売上 ({inputs.year.label})"
    swapXY=true
/>

<DataTable data={status_sales} />

## 月別トレンド

```sql monthly_trend
select
    date_trunc('month', order_date) as month,
    count(*) as order_count,
    round(sum(amount), 2) as revenue
from orders
where status like '${inputs.status.value}'
and date_part('year', order_date)::text like '${inputs.year.value}'
group by 1
order by 1
```

<LineChart
    data={monthly_trend}
    x=month
    y=revenue
    title="月別売上推移 - {inputs.status.label}"
/>

<LineChart
    data={monthly_trend}
    x=month
    y=order_count
    title="月別注文数 - {inputs.status.label}"
/>

## 最近の注文 (TOP 50)

```sql recent_orders
select
    order_id,
    order_date,
    status,
    round(amount, 2) as amount
from orders
where status like '${inputs.status.value}'
and date_part('year', order_date)::text like '${inputs.year.value}'
and case
    when '${inputs.amount_range}' = 'low' then amount < 50
    when '${inputs.amount_range}' = 'medium' then amount between 50 and 200
    when '${inputs.amount_range}' = 'high' then amount > 200
    else true
end
order by order_date desc
limit 50
```

<DataTable data={recent_orders} />

## 注文額別分布

```sql amount_distribution
select
    case
        when amount < 20 then '< $20'
        when amount < 50 then '$20 - $50'
        when amount < 100 then '$50 - $100'
        when amount < 200 then '$100 - $200'
        else '$200+'
    end as value_bucket,
    count(*) as order_count
from orders
where status like '${inputs.status.value}'
and date_part('year', order_date)::text like '${inputs.year.value}'
group by 1
order by 1
```

<BarChart
    data={amount_distribution}
    x=value_bucket
    y=order_count
    title="注文額帯別分布"
/>
