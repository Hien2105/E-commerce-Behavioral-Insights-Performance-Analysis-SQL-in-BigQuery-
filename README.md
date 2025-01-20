# SQL Project 1: Ecommerce 

## 1. Context
I query on the BigQuery environment and the dataset is based on the Google Analytics dataset.

![image](https://github.com/user-attachments/assets/1cac11c2-fd6f-4f23-b1bb-110c32df219c)

![image](https://github.com/user-attachments/assets/f7d02578-b2ab-4625-b942-5d92c70bff51)

![image](https://github.com/user-attachments/assets/8759694e-a9ef-4a47-ad13-47258884dd91)

## 2. Question and output
<details><summary><strong>Query 01: calculate total visit, pageview, transaction for Jan, Feb and March 2017 (order by month)</strong></summary>
<br>
  
```sql
--q1
select
  format_date("%Y%m", parse_date("%Y%m%d", date)) as month,
  sum(totals.visits) as visits,
  sum(totals.pageviews) as pageviews,
  sum(totals.transactions) as transactions,
from `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
where _TABLE_SUFFIX between '0101' and '0331'
group by 1
order by 1;
```
Output

![image](https://github.com/user-attachments/assets/c9362f22-5ea2-482b-a5cc-b8751d0a2008)

</details>
<details><summary><strong>Query 02: Bounce rate per traffic source in July 2017 (Bounce_rate = num_bounce/total_visit) (order by total_visit DESC)</strong></summary>
<br>

```sql
--q2
select
    trafficSource.source as source,
    sum(totals.visits) as total_visits,
    sum(totals.Bounces) as total_no_of_bounces,
    (sum(totals.Bounces)/sum(totals.visits))* 100.00 as bounce_rate
from `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
group by source
order by total_visits DESC;
```
Output

![image](https://github.com/user-attachments/assets/be4136e2-e417-4e11-9613-45dc5c66a383)

</details>
<details><summary><strong>Query 03: Revenue by traffic source by week, by month in June 2017</strong></summary>
<br>

```sql
--q3
with 
month_data as(
  select
    "Month" as time_type,
    format_date("%Y%m", parse_date("%Y%m%d", date)) as month,
    trafficSource.source as source,
    sum(p.productRevenue)/1000000 as revenue
  from `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
    unnest(hits) hits,
    unnest(product) p
  where p.productRevenue is not null
  group by 1,2,3
  order by revenue DESC
),

week_data as(
  select
    "Week" as time_type,
    format_date("%Y%W", parse_date("%Y%m%d", date)) as week,
    trafficSource.source as source,
    sum(p.productRevenue)/1000000 as revenue
  from `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
    unnest(hits) hits,
    unnest(product) p
  where p.productRevenue is not null
  group by 1,2,3
  order by revenue DESC
)

select * from month_data
union all
select * from week_data;
order by time_type
```
Output

![image](https://github.com/user-attachments/assets/a7e2b130-eb4f-4b12-b155-1a2d0db715b5)

</details>
<details><summary><strong>Query 04: Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017.</strong></summary>
<br>

```sql
--q4
with 
purchaser_data as(
  select
      format_date("%Y%m",parse_date("%Y%m%d",date)) as month,
      (sum(totals.pageviews)/count(distinct fullvisitorid)) as avg_pageviews_purchase,
  from `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
    ,unnest(hits) hits
    ,unnest(product) product
  where _table_suffix between '0601' and '0731'
  and totals.transactions>=1
  and product.productRevenue is not null
  group by month
),

non_purchaser_data as(
  select
      format_date("%Y%m",parse_date("%Y%m%d",date)) as month,
      sum(totals.pageviews)/count(distinct fullvisitorid) as avg_pageviews_non_purchase,
  from `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
      ,unnest(hits) hits
    ,unnest(product) product
  where _table_suffix between '0601' and '0731'
  and totals.transactions is null
  and product.productRevenue is null
  group by month
)

select
    pd.*,
    avg_pageviews_non_purchase
from purchaser_data pd
full join non_purchaser_data using(month)
order by pd.month;
```
Output

![image](https://github.com/user-attachments/assets/6f72fcbd-d616-4fa2-af69-63ad076eabbf)

</details>
<details><summary><strong>Query 05: Average number of transactions per user that made a purchase in July 2017</strong></summary>
<br>

```sql
--q5
select
    format_date("%Y%m",parse_date("%Y%m%d",date)) as month,
    sum(totals.transactions)/count(distinct fullvisitorid) as Avg_total_transactions_per_user
from `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
    ,unnest (hits) hits,
    unnest(product) product
where  totals.transactions>=1
and product.productRevenue is not null
group by month;
```
Output

![image](https://github.com/user-attachments/assets/eb3b03a0-393c-4e9b-8dc4-17a30ef92a82)

</details>
<details><summary><strong>Query 06: Average amount of money spent per session. Only include purchaser data in July 2017</strong></summary>
<br>

```sql
--q6
with Raw_data as(
  select
    FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d',date)) as month
    ,totals.visits as visits
    ,product.productRevenue as Revenue
  from `bigquery-public-data.google_analytics_sample.ga_sessions_201707*` 
    ,unnest(hits) as hits
    ,unnest(hits.product) as product
  where totals.transactions is not null and product.productRevenue is not null
)

,Avg_per_visit as(
select 
  month
  ,sum(Revenue) as total_revenue
  ,sum(visits) as total_visit
  ,ROUND((sum(Revenue)/1000000)/sum(visits),2) as avg_revenue_by_user_per_visit
from Raw_data
group by month
)

select month, avg_revenue_by_user_per_visit
from Avg_per_visit;

select
    format_date("%Y%m",parse_date("%Y%m%d",date)) as month,
    ((sum(product.productRevenue)/sum(totals.visits))/power(10,6)) as avg_revenue_by_user_per_visit
from `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
  ,unnest(hits) hits
  ,unnest(product) product
where product.productRevenue is not null
and totals.transactions>=1
group by month;
```
Output

![image](https://github.com/user-attachments/assets/d2a16876-eda8-4366-b2ec-99d6dadefb10)

</details>
<details><summary><strong>Query 07: Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017. Output should show product name and the quantity was ordered.</strong></summary>
<br>

```sql
--q7
select
    product.v2productname as other_purchased_product,
    sum(product.productQuantity) as quantity
from `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
    unnest(hits) as hits,
    unnest(hits.product) as product
where fullvisitorid in (select distinct fullvisitorid
                        from `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
                        unnest(hits) as hits,
                        unnest(hits.product) as product
                        where product.v2productname = "YouTube Men's Vintage Henley"
                        and product.productRevenue is not null)
and product.v2productname != "YouTube Men's Vintage Henley"
and product.productRevenue is not null
group byother_purchased_product
order by quantity desc;

with buyer_list as(
    select
        distinct fullVisitorId  
    from `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
    , unnest(hits) as hits
    , unnest(hits.product) as product
    where product.v2ProductName = "YouTube Men's Vintage Henley"
    and totals.transactions>=1
    and product.productRevenue is not null
)

select
  product.v2ProductName as other_purchased_products,
  sum(product.productQuantity) as quantity
from `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
, unnest(hits) as hits
, unnest(hits.product) as product
join buyer_list using(fullVisitorId)
where product.v2ProductName != "YouTube Men's Vintage Henley"
 and product.productRevenue is not null
group by other_purchased_products
order by quantity DESC;
```
Output

![image](https://github.com/user-attachments/assets/606d986a-0e59-4a6e-9729-afad834b96d7)

</details>
<details><summary><strong>Query 08: Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017. For example, 100% product view then 40% add_to_cart and 10% purchase.
Add_to_cart_rate = number product  add to cart/number product view. Purchase_rate = number product purchase/number product view. The output should be calculated in product level.</strong></summary>
<br>

```sql
--q8
with product_data as(
select
    format_date('%Y%m', parse_date('%Y%m%d',date)) as month,
    count(CasE WHEN eCommerceAction.action_type = '2' THEN product.v2ProductName END) as num_product_view,
    count(CasE WHEN eCommerceAction.action_type = '3' THEN product.v2ProductName END) as num_add_to_cart,
    count(CasE WHEN eCommerceAction.action_type = '6' and product.productRevenue is not null THEN product.v2ProductName END) as num_purchase
from `bigquery-public-data.google_analytics_sample.ga_sessions_*`
,unnest(hits) as hits
,unnest (hits.product) as product
where _table_suffix between '20170101' and '20170331'
and eCommerceAction.action_type in ('2','3','6')
group by month
order by month
)

select
    *,
    round(num_add_to_cart/num_product_view * 100, 2) as add_to_cart_rate,
    round(num_purchase/num_product_view * 100, 2) as purchase_rate
from product_data;
```
Output

![image](https://github.com/user-attachments/assets/e926afad-32c2-4775-b768-ce65dfc4528b)

</details>



