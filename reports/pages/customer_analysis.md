---
sources:
  - motherduck
---
# 👥 顧客分析ダッシュボード

**注意:** このページは`dbt run`で作成される`customers`テーブルを使用します。

## フィルター

<ButtonGroup name=ltv_range>
    <ButtonGroupItem valueLabel="すべて" value="all" default/>
    <ButtonGroupItem valueLabel="$100未満" value="low"/>
    <ButtonGroupItem valueLabel="$100-$500" value="medium"/>
    <ButtonGroupItem valueLabel="$500以上" value="high"/>
</ButtonGroup>

<ButtonGroup name=order_freq>
    <ButtonGroupItem valueLabel="すべて" value="all" default/>
    <ButtonGroupItem valueLabel="1-2回" value="low"/>
    <ButtonGroupItem valueLabel="3-5回" value="medium"/>
    <ButtonGroupItem valueLabel="6回以上" value="high"/>
</ButtonGroup>

---

```sql customer_summary
select
    count(distinct customer_id) as total_customers,
    round(avg(customer_lifetime_value), 2) as avg_lifetime_value,
    round(avg(number_of_orders), 2) as avg_orders_per_customer,
    round(sum(customer_lifetime_value), 2) as total_revenue
from customers
where case
    when '${inputs.ltv_range}' = 'low' then customer_lifetime_value < 100
    when '${inputs.ltv_range}' = 'medium' then customer_lifetime_value between 100 and 500
    when '${inputs.ltv_range}' = 'high' then customer_lifetime_value > 500
    else true
end
and case
    when '${inputs.order_freq}' = 'low' then number_of_orders between 1 and 2
    when '${inputs.order_freq}' = 'medium' then number_of_orders between 3 and 5
    when '${inputs.order_freq}' = 'high' then number_of_orders >= 6
    else true
end
```

## 主要指標

<BigValue
    data={customer_summary}
    value=total_customers
    title="総顧客数"
/>

<BigValue
    data={customer_summary}
    value=avg_lifetime_value
    title="平均生涯価値"
    fmt=usd2
/>

<BigValue
    data={customer_summary}
    value=avg_orders_per_customer
    title="顧客あたり平均注文数"
    fmt=num2
/>

<BigValue
    data={customer_summary}
    value=total_revenue
    title="総収益"
    fmt=usd2
/>

## 生涯価値別トップ顧客 (TOP 20)

```sql top_customers
select
    customer_id,
    first_name || ' ' || last_name as customer_name,
    number_of_orders,
    round(customer_lifetime_value, 2) as lifetime_value,
    first_order,
    most_recent_order
from customers
where case
    when '${inputs.ltv_range}' = 'low' then customer_lifetime_value < 100
    when '${inputs.ltv_range}' = 'medium' then customer_lifetime_value between 100 and 500
    when '${inputs.ltv_range}' = 'high' then customer_lifetime_value > 500
    else true
end
and case
    when '${inputs.order_freq}' = 'low' then number_of_orders between 1 and 2
    when '${inputs.order_freq}' = 'medium' then number_of_orders between 3 and 5
    when '${inputs.order_freq}' = 'high' then number_of_orders >= 6
    else true
end
order by customer_lifetime_value desc
limit 20
```

<DataTable data={top_customers} />

## 顧客生涯価値の分布

```sql ltv_distribution
select
    case
        when customer_lifetime_value < 50 then '< $50'
        when customer_lifetime_value < 100 then '$50 - $100'
        when customer_lifetime_value < 200 then '$100 - $200'
        when customer_lifetime_value < 500 then '$200 - $500'
        else '$500+'
    end as ltv_bucket,
    count(*) as customer_count,
    round(sum(customer_lifetime_value), 2) as total_ltv
from customers
where case
    when '${inputs.order_freq}' = 'low' then number_of_orders between 1 and 2
    when '${inputs.order_freq}' = 'medium' then number_of_orders between 3 and 5
    when '${inputs.order_freq}' = 'high' then number_of_orders >= 6
    else true
end
group by 1
order by 1
```

<BarChart
    data={ltv_distribution}
    x=ltv_bucket
    y=customer_count
    title="生涯価値別顧客数"
/>

<DataTable data={ltv_distribution} />

## 注文頻度分析

```sql order_frequency
select
    number_of_orders,
    count(*) as customer_count,
    round(avg(customer_lifetime_value), 2) as avg_ltv
from customers
where case
    when '${inputs.ltv_range}' = 'low' then customer_lifetime_value < 100
    when '${inputs.ltv_range}' = 'medium' then customer_lifetime_value between 100 and 500
    when '${inputs.ltv_range}' = 'high' then customer_lifetime_value > 500
    else true
end
group by number_of_orders
order by number_of_orders
```

<BarChart
    data={order_frequency}
    x=number_of_orders
    y=customer_count
    title="注文数別顧客数"
/>

<LineChart
    data={order_frequency}
    x=number_of_orders
    y=avg_ltv
    title="注文数と平均生涯価値の関係"
/>

## 顧客獲得推移

```sql acquisition_trend
select
    date_trunc('month', first_order) as month,
    count(*) as new_customers,
    round(avg(customer_lifetime_value), 2) as avg_ltv
from customers
where case
    when '${inputs.ltv_range}' = 'low' then customer_lifetime_value < 100
    when '${inputs.ltv_range}' = 'medium' then customer_lifetime_value between 100 and 500
    when '${inputs.ltv_range}' = 'high' then customer_lifetime_value > 500
    else true
end
group by 1
order by 1
```

<LineChart
    data={acquisition_trend}
    x=month
    y=new_customers
    title="月別新規顧客獲得数"
/>

<LineChart
    data={acquisition_trend}
    x=month
    y=avg_ltv
    title="月別新規顧客の平均生涯価値"
/>

## 顧客セグメント分析

```sql customer_segments
select
    case
        when customer_lifetime_value < 100 then 'Low Value'
        when customer_lifetime_value < 500 then 'Medium Value'
        else 'High Value'
    end as value_segment,
    case
        when number_of_orders <= 2 then 'Low Frequency'
        when number_of_orders <= 5 then 'Medium Frequency'
        else 'High Frequency'
    end as frequency_segment,
    count(*) as customer_count,
    round(avg(customer_lifetime_value), 2) as avg_ltv
from customers
group by 1, 2
order by customer_count desc
```

<DataTable data={customer_segments} />
