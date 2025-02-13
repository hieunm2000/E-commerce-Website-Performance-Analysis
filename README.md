# [SQL] Explore Ecommerce Dataset
## I. Introduction
This project contains an eCommerce dataset that I will explore using SQL on [Google BigQuery](https://cloud.google.com/bigquery). The dataset is based on the Google Analytics public dataset and contains data from an eCommerce website.
## II. Requirements
* [Google Cloud Platform account](https://cloud.google.com)
* Project on Google Cloud Platform
* [Google BigQuery API](https://cloud.google.com/bigquery/docs/enable-transfer-service#:~:text=Enable%20the%20BigQuery%20Data%20Transfer%20Service,-Before%20you%20can&text=Open%20the%20BigQuery%20Data%20Transfer,Click%20the%20ENABLE%20button.) enabled
* [SQL query editor](https://cloud.google.com/monitoring/mql/query-editor) or IDE
## III. Dataset Access
The eCommerce dataset is stored in a public Google BigQuery dataset. To access the dataset, follow these steps:
* Log in to your Google Cloud Platform account and create a new project.
* Navigate to the BigQuery console and select your newly created project.
* In the navigation panel, select "Add Data" and then "Search a project".
* Enter the project ID **"bigquery-public-data.google_analytics_sample.ga_sessions"** and click "Enter".
* Click on the **"ga_sessions_"** table to open it.
## IV. Exploring the Dataset
In this project, I will write 08 query in Bigquery base on Google Analytics dataset

### Query 01: Calculate total visit, pageview, transaction and revenue for January, February and March 2017 order by month
* SQL code

![image](https://github.com/user-attachments/assets/ddb9bc18-6131-411d-b38c-f24757b3ef02)


* Query results

![image](https://github.com/user-attachments/assets/b9ac3046-d7e4-4873-96aa-de45924b243d)

### Query 02: Bounce rate per traffic source in July 2017
* SQL code

![image](https://github.com/user-attachments/assets/30d2c0fe-f111-4c86-a1ad-62d060cbb511)


* Query results

![image](https://github.com/user-attachments/assets/4976913e-dfda-45dc-bb4f-0ddcc8a2e915)


### Query 3: Revenue by traffic source by week, by month in June 2017
* SQL code

![image](https://github.com/user-attachments/assets/90522a5e-233d-4021-9089-59f943a50cdd)


* Query results

![image](https://github.com/user-attachments/assets/752f3592-e0bb-4031-b67c-1a57217e8833)

### Query 04: Average number of product pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017
* SQL code

![image](https://github.com/user-attachments/assets/fafb1b5c-b091-4725-89ab-88ff4430d2d7)

* Query results

![image](https://github.com/user-attachments/assets/2137a364-bc8c-46bb-b756-744a5a22a01f)


### Query 05: Average number of transactions per user that made a purchase in July 2017
* SQL code

![image](https://github.com/user-attachments/assets/1badbb40-e7c5-44dc-b50c-7445f23092e0)


* Query results

![image](https://github.com/user-attachments/assets/43c0764d-ea9d-48b2-a977-5eb690235aa9)


### Query 06: Average amount of money spent per session. Only include purchaser data in July 2017
* SQL code

![image](https://github.com/user-attachments/assets/a357892d-e0ba-4728-afed-13dcab300ce5)


* Query results

![image](https://github.com/user-attachments/assets/f31d0b68-6c2f-47f7-8a55-aa64b3783dbd)


### Query 07: Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017. Output should show product name and the quantity was ordered.
* SQL code

![image](https://github.com/user-attachments/assets/e677b33e-250a-4533-b808-9dc287210c73)


* Query results

![image](https://github.com/user-attachments/assets/31839964-1b3e-44a5-95b0-4d4d690b6826)



  ### Query 08: C"Query 08: Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017.
  * SQL code

```

with xem_san_pham AS (
SELECT  EXTRACT(MONTH FROM PARSE_DATE('%Y%m%d', date)) AS month, 
        COUNT(*) AS num_product_view
FROM  `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
UNNEST (hits) AS hits,
UNNEST (hits.product) AS product
WHERE
      hits.eCommerceAction.action_type = '2'
      AND EXTRACT(YEAR FROM PARSE_DATE('%Y%m%d', date)) = 2017 
      AND EXTRACT(MONTH FROM PARSE_DATE('%Y%m%d', date)) IN (01, 02, 03) 
GROUP BY month
),

add_to_cart AS(
  SELECT EXTRACT(MONTH FROM PARSE_DATE('%Y%m%d', date)) AS month,
        COUNT(*) AS num_addtocart
FROM  `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
UNNEST (hits) AS hits,
UNNEST (hits.product) AS product
WHERE
      hits.eCommerceAction.action_type = '3'
      AND EXTRACT(YEAR FROM PARSE_DATE('%Y%m%d', date)) = 2017
      AND EXTRACT(MONTH FROM PARSE_DATE('%Y%m%d', date)) IN (01, 02, 03) 
GROUP BY month
),

mua_hang AS (
SELECT EXTRACT(MONTH FROM PARSE_DATE('%Y%m%d', date)) AS month,
        COUNT(*) AS num_purchase
FROM  `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
UNNEST (hits) AS hits,
UNNEST (hits.product) AS product
WHERE
      hits.eCommerceAction.action_type = '6'
      AND EXTRACT(YEAR FROM PARSE_DATE('%Y%m%d', date)) = 2017 
      AND EXTRACT(MONTH FROM PARSE_DATE('%Y%m%d', date)) IN (01, 02, 03) 
GROUP BY month
)

SELECT
  xem_san_pham.month,
  num_product_view AS num_product_view,
  num_addtocart AS num_addtocart,
  num_purchase AS num_purchase,
  ROUND((num_addtocart / num_product_view) * 100, 2) AS add_to_cart_rate,
  ROUND((num_purchase / num_product_view) * 100, 2) AS purchase_rate
FROM xem_san_pham

LEFT JOIN add_to_cart
ON xem_san_pham.month = add_to_cart.month 

LEFT JOIN mua_hang
ON xem_san_pham.month = mua_hang.month 

ORDER BY month;

with
product_view as(
  SELECT
    format_date("%Y%m", parse_date("%Y%m%d", date)) as month,
    count(product.productSKU) as num_product_view
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
  , UNNEST(hits) AS hits
  , UNNEST(hits.product) as product
  WHERE _TABLE_SUFFIX BETWEEN '20170101' AND '20170331'
  AND hits.eCommerceAction.action_type = '2'
  GROUP BY 1
),

add_to_cart as(
  SELECT
    format_date("%Y%m", parse_date("%Y%m%d", date)) as month,
    count(product.productSKU) as num_addtocart
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
  , UNNEST(hits) AS hits
  , UNNEST(hits.product) as product
  WHERE _TABLE_SUFFIX BETWEEN '20170101' AND '20170331'
  AND hits.eCommerceAction.action_type = '3'
  GROUP BY 1
),

purchase as(
  SELECT
    format_date("%Y%m", parse_date("%Y%m%d", date)) as month,
    count(product.productSKU) as num_purchase
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
  , UNNEST(hits) AS hits
  , UNNEST(hits.product) as product
  WHERE _TABLE_SUFFIX BETWEEN '20170101' AND '20170331'
  AND hits.eCommerceAction.action_type = '6'
  and product.productRevenue is not null   
  group by 1
)

select
    pv.*,
    num_addtocart,
    num_purchase,
    round(num_addtocart*100/num_product_view,2) as add_to_cart_rate,
    round(num_purchase*100/num_product_view,2) as purchase_rate
from product_view pv
left join add_to_cart a on pv.month = a.month
left join purchase p on pv.month = p.month
order by pv.month;

```
* Query results

![image](https://user-images.githubusercontent.com/101726623/235148311-a2d83174-9bf3-43e3-aed1-47030af40b3b.png)

## V. Conclusion
* In conclusion, my exploration of the eCommerce dataset using SQL on Google BigQuery based on the Google Analytics dataset has revealed several interesting insights.
* By exploring eCommerce dataset, I have gained valuable information about total visits, pageview, transactions, bounce rate, and revenue per traffic source,.... which could inform future business decisions.
* To deep dive into the insights and key trends, the next step will visualize the data with some software like Power BI,Tableau,...
* **Overall**, this project has demonstrated the power of using SQL and big data tools like Google BigQuery to gain insights into large datasets.
