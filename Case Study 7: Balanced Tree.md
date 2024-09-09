## 🌲 Case Study #7: Balanced Tree

	




<img src="https://github.com/katiehuangx/8-Week-SQL-Challenge/assets/81607668/8ada3c0c-e90a-47a7-9a5c-8ffd6ee3eef8" alt="Image" width="500" height="520">

## 📚 Table of Contents
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Question and Solution](#question-and-solution)

Please note that all the information regarding the case study has been sourced from the following link: [here](https://8weeksqlchallenge.com/case-study-7/). 

***

## Business Task

Balanced Tree Clothing Company prides themselves on providing an optimised range of clothing and lifestyle wear for the modern adventurer!

Danny, the CEO of this trendy fashion company has asked you to assist the team’s merchandising teams **analyse their sales performance and generate a basic financial report** to share with the wider business.

## Entity Relationship Diagram

![image](https://github.com/user-attachments/assets/b29e6c09-8919-42c5-a92c-ac9935bd53c2)



**Table 1: `product_details`**

![image](https://github.com/user-attachments/assets/53e0f53b-e8eb-49d0-a5cd-d3d5e2f04b68)



**Table 2: `sales`**

![image](https://github.com/user-attachments/assets/21e2cfa3-9d64-471a-87fb-98872086f362)



**Table 3: `product_hierarchy`**

|id|parent_id|level_text|level_name|
|:----|:----|:----|:----|
|1|null|Womens|Category|
|2|null|Mens|Category|
|3|1|Jeans|Segment|
|4|1|Jacket|Segment|
|5|2|Shirt|Segment|
|6|2|Socks|Segment|
|7|3|Navy Oversized|Style|
|8|3|Black Straight|Style|
|9|3|Cream Relaxed|Style|
|10|4|Khaki Suit|Style|

**Table 4: `product_prices`**

|id|product_id|price|
|:----|:----|:----|
|7|c4a632|13|
|8|e83aa3|32|
|9|e31d39|10|
|10|d5e9a6|23|
|11|72f5d4|19|
|12|9ec847|54|
|13|5d267b|40|
|14|c8d436|10|
|15|2a2353|57|
|16|f084eb|36|

***

## Question and Solution

## 📈 A. High Level Sales Analysis

**1. What was the total quantity sold for all products?**

```sql
SELECT 
  product.product_name, 
  SUM(sales.qty) AS total_quantity
FROM balanced_tree.sales
INNER JOIN balanced_tree.product_details AS product
	ON sales.prod_id = product.product_id
GROUP BY product.product_name;
```

**Answer:**

<img width="400" alt="image" src="https://github.com/user-attachments/assets/8cbda0bf-12c3-48bd-a1c2-b86ccc74f040">

***

**2. What is the total generated revenue for all products before discounts?**

```sql

SELECT 
  product.product_name, 
   SUM(sales.qty)* SUM(sales.price) AS total_generated_revenue
FROM balanced_tree.sales
INNER JOIN balanced_tree.product_details AS product
	ON sales.prod_id = product.product_id
GROUP BY product.product_name;
```

**Answer:**

<img width="400" alt="image" src="https://github.com/user-attachments/assets/c90b92d9-342c-4326-bd29-2cb96a26070b">

***

**3. What was the total discount amount for all products?**

```sql
SELECT 
  product.product_name, 
  SUM(sales.qty * sales.price * sales.discount/100) AS total_discount
FROM balanced_tree.sales
INNER JOIN balanced_tree.product_details AS product
	ON sales.prod_id = product.product_id
GROUP BY product.product_name;

```

**Answer:**

<img width="400" alt="image" src="https://github.com/user-attachments/assets/6022de15-3ef3-4ebb-a67e-1be6b30d94f6">

***

## 🧾 B. Transaction Analysis

**1. How many unique transactions were there?**

```sql
SELECT COUNT(DISTINCT txn_id) AS transaction_count
FROM sales;
```

**Answer:**

![image](https://github.com/user-attachments/assets/3480e418-33aa-4d95-95b7-ae7fabdc1730)


***

**2. What is the average unique products purchased in each transaction?**

```sql
SELECT ROUND(SUM(unique_product_ordered)/COUNT(txn_id)) AS avg_unique_products
FROM (
  SELECT 
    txn_id, 
    COUNT(DISTINCT prod_id) AS unique_product_ordered
  FROM sales
  GROUP BY txn_id
) AS total_unique_products ;
```

**Answer:**

![image](https://github.com/user-attachments/assets/d1c59d53-547b-4c9a-a6dc-4da5be1a65b8)


***

**3. What are the 25th, 50th and 75th percentile values for the revenue per transaction?**

```sql
WITH revenue_cte AS (
  SELECT 
    txn_id, 
    SUM(price * qty) AS revenue
  FROM balanced_tree.sales
  GROUP BY txn_id
)

SELECT
  PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY revenue) AS median_25th,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY revenue) AS median_50th,
PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY revenue) AS median_75th
FROM revenue_cte;
```

**Answer:**

|median_25th|median_50th|median_75th|
|:----|:----|:----|
|375.75|509.5|647|

***

**4. What is the average discount value per transaction?**

```sql
SELECT ROUND(AVG(discount_amt)) AS avg_discount
FROM (
  SELECT 
	  txn_id,
    SUM(qty * price * discount/100) AS discount_amt
  FROM balanced_tree.sales
  GROUP BY txn_id
) AS discounted_value
```

**Answer:**

|avg_discount|
|:----|
|60|

**5. What is the percentage split of all transactions for members vs non-members?**

```sql
WITH transactions_cte AS (
  SELECT
    member,
    COUNT(DISTINCT txn_id) AS transactions
  FROM balanced_tree.sales
  GROUP BY member
)

SELECT
	member,
  transactions,
  ROUND(100 * transactions
    /(SELECT SUM(transactions) 
      FROM transactions_cte)) AS percentage
FROM transactions_cte
GROUP BY member, transactions;
```

**Answer:**

Members have a transaction count at 60% compared to than non-members who account for only 40% of the transactions.

|member|transactions|percentage|
|:----|:----|:----|
|false|995|40|
|true|1505|60|

***

**6. What is the average revenue for member transactions and non-member transactions?**

```sql
WITH revenue_cte AS (
  SELECT
    member,
  	txn_id,
    SUM(price * qty) AS revenue
  FROM balanced_tree.sales
  GROUP BY member, txn_id
)

SELECT
	member,
  ROUND(AVG(revenue),2) AS avg_revenue
FROM revenue_cte
GROUP BY member;
```

**Answer:**

The average revenue per transaction for members is only $1.23 higher compared to non-members.

|member|avg_revenue|
|:----|:----|
|false|515.04|
|true|516.27|

***

## 👚 C. Product Analysis

**1. What are the top 3 products by total revenue before discount?**

```sql
SELECT 
  product.product_id,
  product.product_name, 
  SUM(sales.qty) * SUM(sales.price) AS total_revenue
FROM balanced_tree.sales
INNER JOIN balanced_tree.product_details AS product
	ON sales.prod_id = product.product_id
GROUP BY product.product_id, product.product_name
ORDER BY total_revenue DESC
LIMIT 3;
```

**Answer:**

|product_id|product_name|total_revenue|
|:----|:----|:----|
|2a2353|Blue Polo Shirt - Mens|276022044|
|9ec847|Grey Fashion Jacket - Womens|266862600|
|5d267b|White Tee Shirt - Mens|192736000|

***

**2. What is the total quantity, revenue and discount for each segment?**

```sql
SELECT 
  product.segment_id,
  product.segment_name, 
  SUM(sales.qty) AS total_quantity,
  SUM(sales.qty * sales.price) AS total_revenue,
  SUM((sales.qty * sales.price) * sales.discount/100) AS total_discount
FROM balanced_tree.sales
INNER JOIN balanced_tree.product_details AS product
	ON sales.prod_id = product.product_id
GROUP BY product.segment_id, product.segment_name;
```

**Answer:**

|segment_id|segment_name|total_quantity|total_revenue|total_discount|
|:----|:----|:----|:----|:----|
|4|Jacket|11385|366983|42451|
|6|Socks|11217|307977|35280|
|5|Shirt|11265|406143|48082|
|3|Jeans|11349|208350|23673|

***

**3. What is the top selling product for each segment?**

```sql
WITH top_selling_cte AS ( 
  SELECT 
    product.segment_id,
    product.segment_name, 
    product.product_id,
    product.product_name,
    SUM(sales.qty) AS total_quantity,
    RANK() OVER (
      PARTITION BY segment_id 
      ORDER BY SUM(sales.qty) DESC) AS ranking
  FROM balanced_tree.sales
  INNER JOIN balanced_tree.product_details AS product
    ON sales.prod_id = product.product_id
  GROUP BY 
    product.segment_id, product.segment_name, product.product_id, product.product_name
)

SELECT 
  segment_id,
  segment_name, 
  product_id,
  product_name,
  total_quantity
FROM top_selling_cte
WHERE ranking = 1;
```

**Answer:**

|segment_id|segment_name|product_id|product_name|total_quantity|
|:----|:----|:----|:----|:----|
|3|Jeans|c4a632|Navy Oversized Jeans - Womens|3856|
|4|Jacket|9ec847|Grey Fashion Jacket - Womens|3876|
|5|Shirt|2a2353|Blue Polo Shirt - Mens|3819|
|6|Socks|f084eb|Navy Solid Socks - Mens|3792|

***

**4. What is the total quantity, revenue and discount for each category?**

```sql
SELECT 
  product.category_id,
  product.category_name, 
  SUM(sales.qty) AS total_quantity,
  SUM(sales.qty * sales.price) AS total_revenue,
  SUM((sales.qty * sales.price) * sales.discount/100) AS total_discount
FROM balanced_tree.sales
INNER JOIN balanced_tree.product_details AS product
	ON sales.prod_id = product.product_id
GROUP BY product.category_id, product.category_name
ORDER BY product.category_id;
```

**Answer:**

|category_id|category_name|total_quantity|total_revenue|total_discount|
|:----|:----|:----|:----|:----|
|1|Womens|22734|575333|66124|
|2|Mens|22482|714120|83362|

***

**5. What is the top selling product for each category?**

```sql
WITH top_selling_cte AS ( 
  SELECT 
    product.category_id,
    product.category_name, 
    product.product_id,
    product.product_name,
    SUM(sales.qty) AS total_quantity,
    RANK() OVER (
      PARTITION BY product.category_id 
      ORDER BY SUM(sales.qty) DESC) AS ranking
  FROM balanced_tree.sales
  INNER JOIN balanced_tree.product_details AS product
    ON sales.prod_id = product.product_id
  GROUP BY 
    product.category_id, product.category_name, product.product_id, product.product_name
)

SELECT 
  category_id,
  category_name, 
  product_id,
  product_name,
  total_quantity
FROM top_selling_cte
WHERE ranking = 1;
```

**Answer:**

|category_id|category_name|product_id|product_name|total_quantity|
|:----|:----|:----|:----|:----|
|1|Womens|9ec847|Grey Fashion Jacket - Womens|3876|
|2|Mens|2a2353|Blue Polo Shirt - Mens|3819|

***

**6. What is the percentage split of revenue by product for each segment?**

**Answer:**

***

**7. What is the percentage split of revenue by segment for each category?**

**Answer:**

***

**8. What is the percentage split of total revenue by category?**

**Answer:**

***

**9. What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)**

**Answer:**

***

**10. What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?**

**Answer:**

***

## 📝 Reporting Challenge

Write a single SQL script that combines all of the previous questions into a scheduled report that the Balanced Tree team can run at the beginning of each month to calculate the previous month’s values.

Imagine that the Chief Financial Officer (which is also Danny) has asked for all of these questions at the end of every month.

He first wants you to generate the data for January only - but then he also wants you to demonstrate that you can easily run the samne analysis for February without many changes (if at all).

Feel free to split up your final outputs into as many tables as you need - but be sure to explicitly reference which table outputs relate to which question for full marks :)

***

## 💡 Bonus Challenge

Use a single SQL query to transform the `product_hierarchy` and `product_prices` datasets to the `product_details` table.

Hint: you may want to consider using a recursive CTE to solve this problem!

***
