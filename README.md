# SQL Project: E-commerce Behavioral Insights & Performance Analysis with SQL in BigQuery

## 1. Situation
An e-commerce business wanted to optimize its marketing strategies, improve conversion rates, and enhance user experience. To achieve this, understanding customer behavior, traffic sources, and purchasing patterns was essential.

## 2. Task
Using SQL in Google BigQuery, I conducted a comprehensive analysis of website traffic, user engagement, and product performance to uncover insights that drive revenue growth and customer retention.

Dataset: Google Analytics Public Dataset

![](https://github.com/Hien2105/photo/blob/main/Sql%202.png?raw=true)

Dataset dictionary: 

![](https://github.com/Hien2105/photo/blob/main/Sql%201.png?raw=true)

## 3. Action
### Action Breakdown
As part of this project, I analyzed user behavior, marketing performance, and purchase trends using SQL in BigQuery. My goal was to uncover key traffic patterns, evaluate engagement metrics, and identify the factors driving conversions. By breaking down data across traffic sources, user interactions, and transaction behavior, I extracted actionable insights to enhance customer experience and optimize revenue.  

The table below outlines the key actions I took in this analysis:  

| #  | Category                                | Description  |
|----|-----------------------------------------|-------------|
| 1  | **Traffic & Engagement Analysis**      | Measured total visits, page views, and transactions in Q1 2017 to identify key traffic trends and seasonal patterns. |
| 2  | **Marketing Effectiveness**            | Evaluated bounce rates across traffic sources in July 2017 to pinpoint ineffective channels and optimize landing pages. |
| 3  | **Revenue Breakdown**                   | Analyzed revenue by traffic source on a weekly and monthly basis in June 2017 to assess the best-performing acquisition channels. |
| 4  | **User Behavior Comparison**            | Compared the browsing patterns of purchasers vs. non-purchasers in June & July 2017 to identify key engagement drivers. |
| 5  | **Customer Loyalty & Spending Patterns** | Measured transaction frequency per user and average spending per session in July 2017 to gauge purchase consistency and spending habits. |
| 6  | **Product Affinity & Cross-Selling**    | Identified frequently co-purchased products with YouTube Men's Vintage Henley to uncover bundling and recommendation opportunities. |
| 7  | **Conversion Funnel Optimization**      | Built a cohort analysis to track product view-to-purchase conversion rates in Q1 2017, revealing key drop-off points in the buying journey. |

<details><summary><strong>1. Traffic & Engagement Analysis</strong></summary>
<br>
Measured total visits, page views, and transactions in Q1 2017 to identify key traffic trends and seasonal patterns.

```sql
--q1 calculate total visit, pageview, transaction for Jan, Feb and March 2017 (order by month)
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
Query Result:

| Month  | Visits | Pageviews | Transactions |
|--------|--------|-----------|--------------|
| 201701 | 64,694 | 257,708   | 713          |
| 201702 | 62,192 | 233,373   | 733          |
| 201703 | 69,931 | 259,522   | 993          |


</details>
<details><summary><strong>2. Marketing Effectiveness</strong></summary>
<br>
Evaluated bounce rates across traffic sources in July 2017 to pinpoint ineffective channels and optimize landing pages.

  
```sql
--q2 Bounce rate per traffic source in July 2017 (Bounce_rate = num_bounce/total_visit) (order by total_visit DESC)
select
    trafficSource.source as source,
    sum(totals.visits) as total_visits,
    sum(totals.Bounces) as total_no_of_bounces,
    (sum(totals.Bounces)/sum(totals.visits))* 100.00 as bounce_rate
from `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
group by source
order by total_visits DESC;
```
Query Result:

| Source | Total Visits | Total Bounces | Bounce Rate (%) |
|--------|-------------|--------------|---------------|
| google | 38,400 | 19,798 | 51.56% |
| (direct) | 19,891 | 8,606 | 43.27% |
| youtube.com | 6,351 | 4,238 | 66.73% |
| analytics.google.com | 1,972 | 1,064 | 53.96% |
| Partners | 1,788 | 936 | 52.35% |
| m.facebook.com | 669 | 430 | 64.28% |
| google.com | 368 | 183 | 49.73% |
| dfa | 302 | 124 | 41.06% |
| sites.google.com | 230 | 97 | 42.17% |
| facebook.com | 191 | 102 | 53.40% |
| reddit.com | 189 | 54 | 28.57% |
| ... | ... | ... | ... |


</details>
<details><summary><strong>3. Revenue Breakdown</strong></summary>
<br>
Analyzed revenue by traffic source on a weekly and monthly basis in June 2017 to assess the best-performing acquisition channels.

```sql
--q3 Revenue by traffic source by week, by month in June 2017
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
Query Result:

| Time Type | Time   | Source  | Revenue ($) |
|-----------|--------|---------|-------------|
| Month     | 201706 | (direct) | 97,333.62  |
| Week      | 201724 | (direct) | 30,908.91  |
| Week      | 201725 | (direct) | 27,295.32  |
| Month     | 201706 | google   | 18,757.18  |
| Week      | 201723 | (direct) | 17,325.68  |
| Week      | 201726 | (direct) | 14,914.81  |
| Week      | 201724 | google   | 9,217.17   |
| Week      | 201722 | (direct) | 6,888.90   |
| Week      | 201726 | google   | 5,330.57   |
| Week      | 201722 | google   | 2,119.39   |
| Week      | 201723 | google   | 1,083.95   |
| Week      | 201725 | google   | 1,006.10   |

</details>
<details><summary><strong>4. User Behavior Comparison</strong></summary>
<br>
Compared the browsing patterns of purchasers vs. non-purchasers in June & July 2017 to identify key engagement drivers.
  
```sql
--q4 Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017.
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
Query Result:

| Month  | Avg Pageviews (Purchase) | Avg Pageviews (Non-Purchase) |
|--------|-------------------------:|-----------------------------:|
| 201706 | 94.02                    | 316.87                      |
| 201707 | 124.24                   | 334.06                      |


</details>
<details><summary><strong>5. Customer Loyalty & Spending Patterns</strong></summary>
<br>
Measured transaction frequency per user and average spending per session in July 2017 to gauge purchase consistency and spending habits.
  
```sql
--q5 Average number of transactions per user that made a purchase in July 2017
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
Query Result:

| Month  | Avg Total Transactions per User |
|--------|--------------------------------:|
| 201707 | 4.16                            |

```sql
--q6 Average amount of money spent per session. Only include purchaser data in July 2017
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
Query Result:

| Month  | Avg Revenue Per Visit (USD) |
|--------|----------------------------:|
| 201707 | 43.86                       |


</details>



</details>
<details><summary><strong>6. Product Affinity & Cross-Selling </strong></summary>
<br>
Identified frequently co-purchased products with YouTube Men's Vintage Henley to uncover bundling and recommendation opportunities.
  
```sql
--q7 Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017.
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
Query Result:

| Product Name                                      | Quantity |
|--------------------------------------------------|---------:|
| Google Sunglasses                                | 20       |
| Google Women's Vintage Hero Tee Black           | 7        |
| SPF-15 Slim & Slender Lip Balm                  | 6        |
| Google Women's Short Sleeve Hero Tee Red Heather | 4        |
| YouTube Men's Fleece Hoodie Black               | 3        |
| Google Men's Short Sleeve Badge Tee Charcoal    | 3        |


</details>
<details><summary><strong>7. Conversion Funnel Optimization </strong></summary>
<br>
Built a cohort analysis to track product view-to-purchase conversion rates in Q1 2017, revealing key drop-off points in the buying journey.

```sql
--q8 Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017. 
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
Query Result:

| Month  | Product Views | Add to Cart | Purchases | Add-to-Cart Rate (%) | Purchase Rate (%) |
|--------|--------------|-------------|-----------|----------------------|------------------:|
| 201701 | 25,787       | 7,342       | 2,143     | 28.47                | 8.31             |
| 201702 | 21,489       | 7,360       | 2,060     | 34.25                | 9.59             |
| 201703 | 23,549       | 8,782       | 2,977     | 37.29                | 12.64            |


</details>

 

## 4. Result (Key Insights & recommendations)
### Key Insights

| #  | Category                          | Insight Summary |
|----|----------------------------------|------------------------------------------------------|
| 1  | Traffic & Engagement Analysis    | Visits were stable in Q1 2017, while transactions rose, improving the conversion rate from 1.10% to 1.42%. |
| 2  | Marketing Effectiveness          | Google had high visits but a 51.56% bounce rate, while YouTube had the worst engagement (66.73% bounce rate). Reddit had the lowest (28.57%), showing better content relevance. |
| 3  | Revenue Breakdown                | In June 2017, direct traffic drove the most revenue ($97K), far exceeding Google ($18K). Weekly trends showed direct traffic consistently outperforming Google. |
| 4  | User Behavior Comparison         | Purchasers viewed fewer pages (94–124) than non-purchasers (317–334), suggesting difficulties in product discovery or decision-making. |
| 5  | Customer Loyalty & Spending      | In July 2017, users averaged 4.16 transactions and spent $43.86 per visit, indicating strong engagement. |
| 6  | Product Affinity & Cross-Selling | Google Sunglasses (20 units) led sales, followed by apparel, showing demand for branded accessories. |
| 7  | Conversion Funnel Optimization   | Add-to-cart (28.47% → 37.29%) and purchase rates (8.31% → 12.64%) steadily improved in Q1 2017, signaling better conversion efficiency. |


---

### Recommendations

| #  | Category | Recommended Actions |
|----|--------------------------|----------------------------------------------------------------|
| 1  | Traffic Optimization | Enhance Google ad targeting to increase revenue share from search traffic. |
| 2  | Landing Page & CRO | Improve landing pages for YouTube-driven traffic to reduce bounce rates. |
| 3  | User Experience (UX) Enhancement | Simplify navigation & optimize checkout to help non-buyers (316+ pageviews) convert faster. |
| 4  | Conversion Funnel Refinement | Address drop-offs in the buying journey, possibly by optimizing the cart and checkout flow. |









