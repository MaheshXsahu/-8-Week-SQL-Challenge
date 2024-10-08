# Case Study #8: Fresh Segments


## 📚 Table of Contents
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Question and Solution](#question-and-solution)

Please note that all the information regarding the case study has been sourced from the following link: [here](https://8weeksqlchallenge.com/case-study-8/). 

***

## Business Task

Fresh Segments is a digital marketing agency that helps other businesses analyse trends in online ad click behaviour for their unique customer base.

Clients share their customer lists with the Fresh Segments team who then aggregate interest metrics and generate a single dataset worth of metrics for further analysis.

In particular - the composition and rankings for different interests are provided for each client showing the proportion of their customer list who interacted with online assets related to each interest for each month.

Danny has asked for your assistance to analyse aggregated metrics for an example client and provide some high level insights about the customer list and their interests.

***

## Entity Relationship Diagram

**Table: `interest_map`**
<kbd><img width="970" alt="image" src="https://user-images.githubusercontent.com/81607668/138912360-49996d84-23e4-40a7-98a1-9a8e9341b492.png"></kbd>

| id | interest_name             | interest_summary                                                                   | created_at              | last_modified           |
|----|---------------------------|------------------------------------------------------------------------------------|-------------------------|-------------------------|
| 1  | Fitness Enthusiasts       | Consumers using fitness tracking apps and websites.                                | 2016-05-26T14:57:59.000 | 2018-05-23T11:30:12.000 |
| 2  | Gamers                    | Consumers researching game reviews and cheat codes.                                | 2016-05-26T14:57:59.000 | 2018-05-23T11:30:12.000 |
| 3  | Car Enthusiasts           | Readers of automotive news and car reviews.                                        | 2016-05-26T14:57:59.000 | 2018-05-23T11:30:12.000 |
| 4  | Luxury Retail Researchers | Consumers researching luxury product reviews and gift ideas.                       | 2016-05-26T14:57:59.000 | 2018-05-23T11:30:12.000 |
| 5  | Brides & Wedding Planners | People researching wedding ideas and vendors.                                      | 2016-05-26T14:57:59.000 | 2018-05-23T11:30:12.000 |
| 6  | Vacation Planners         | Consumers reading reviews of vacation destinations and accommodations.             | 2016-05-26T14:57:59.000 | 2018-05-23T11:30:13.000 |
| 7  | Motorcycle Enthusiasts    | Readers of motorcycle news and reviews.                                            | 2016-05-26T14:57:59.000 | 2018-05-23T11:30:13.000 |
| 8  | Business News Readers     | Readers of online business news content.                                           | 2016-05-26T14:57:59.000 | 2018-05-23T11:30:12.000 |
| 12 | Thrift Store Shoppers     | Consumers shopping online for clothing at thrift stores and researching locations. | 2016-05-26T14:57:59.000 | 2018-03-16T13:14:00.000 |
| 13 | Advertising Professionals | People who read advertising industry news.                                         | 2016-05-26T14:57:59.000 | 2018-05-23T11:30:12.000 |

**Table: `interest_metrics`**

| month | year | month_year | interest_id | composition | index_value | ranking | percentile_ranking |
|-------|------|------------|-------------|-------------|-------------|---------|--------------------|
| 7     | 2018 | Jul-18     | 32486       | 11.89       | 6.19        | 1       | 99.86              |
| 7     | 2018 | Jul-18     | 6106        | 9.93        | 5.31        | 2       | 99.73              |
| 7     | 2018 | Jul-18     | 18923       | 10.85       | 5.29        | 3       | 99.59              |
| 7     | 2018 | Jul-18     | 6344        | 10.32       | 5.1         | 4       | 99.45              |
| 7     | 2018 | Jul-18     | 100         | 10.77       | 5.04        | 5       | 99.31              |
| 7     | 2018 | Jul-18     | 69          | 10.82       | 5.03        | 6       | 99.18              |
| 7     | 2018 | Jul-18     | 79          | 11.21       | 4.97        | 7       | 99.04              |
| 7     | 2018 | Jul-18     | 6111        | 10.71       | 4.83        | 8       | 98.9               |
| 7     | 2018 | Jul-18     | 6214        | 9.71        | 4.83        | 8       | 98.9               |
| 7     | 2018 | Jul-18     | 19422       | 10.11       | 4.81        | 10      | 98.63              |

***

## Question and Solution

Please join me in executing the queries using PostgreSQL on [DB Fiddle](https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/17). It would be great to work together on the questions!

If you have any questions, reach out to me on [LinkedIn](https://www.linkedin.com/in/katiehuangx/).

## 🧼 A. Data Exploration and Cleansing

**1. Update the `fresh_segments.interest_metrics` table by modifying the `month_year` column to be a `date` data type with the start of the month**

```sql
ALTER TABLE fresh_segments.interest_metrics
ALTER month_year TYPE DATE USING month_year::DATE;
```

<kbd><img width="970" alt="image" src="https://github.com/user-attachments/assets/ecce0d9f-078f-4dc0-909d-f6b318435c29"></kbd>

***
  
**2. What is count of records in the `fresh_segments.interest_metrics` for each `month_year` value sorted in chronological order (earliest to latest) with the `null` values appearing first?**

```sql
SELECT 
  month_year, COUNT(*)
FROM fresh_segments.interest_metrics
GROUP BY month_year
ORDER BY month_year NULLS FIRST;
```
<kbd><img width="291" alt="image" src="https://github.com/user-attachments/assets/5f5c5a14-a3b6-4cba-9711-2901da22a7d9"></kbd>

**3. What do you think we should do with these `null` values in the `fresh_segments.interest_metrics`?**

The `null` values appear in `_month`, `_year`, `month_year`, and `interest_id`. The corresponding values in `composition`, `index_value`, `ranking`, and `percentile_ranking` fields are not meaningful without the specific information on `interest_id` and dates. 

Before dropping the values, it would be useful to find out the percentage of `null` values.

```sql
SELECT 
  ROUND(100 * (SUM(CASE WHEN interest_id IS NULL THEN 1 END) * 1.0 /
    COUNT(*)),2) AS null_perc
FROM fresh_segments.interest_metrics
```

<kbd><img width="100" alt="image" src="https://user-images.githubusercontent.com/81607668/138892507-5b89eba8-45c7-4edf-9c05-42347f47c746.png"></kbd>

The percentage of null values is 8.36% which is less than 10%, hence I would suggest to drop all the `null` values.

```sql
DELETE FROM fresh_segments.interest_metrics
WHERE interest_id IS NULL;

-- Run again to confirm that there are no null values.
SELECT 
  ROUND(100 * (SUM(CASE WHEN interest_id IS NULL THEN 1 END) * 1.0 /
    COUNT(*)),2) AS null_perc
FROM fresh_segments.interest_metrics
```

<kbd><img width="120" alt="image" src="https://github.com/user-attachments/assets/e9e4956e-c49e-40a9-9744-702b26e2da30"></kbd>

Confirmed that there are no `null` values in `fresh_segments.interest_metrics`.

***
  
**4. How many `interest_id` values exist in the `fresh_segments.interest_metrics` table but not in the `fresh_segments.interest_map` table? What about the other way around?**.

Before we perform the actual Query for the solution we need to convert the interest_id to INTEGER DATA TYPE:

```sql
ALTER TABLE interest_metrics
ALTER COLUMN interest_id TYPE INTEGER
USING CAST(interest_id AS INTEGER);
```
 The Final Query :

```sql
SELECT 
  COUNT(DISTINCT map.id) AS map_id_count,
  COUNT(DISTINCT metrics.interest_id) AS metrics_id_count,
  SUM(CASE WHEN map.id is NULL THEN 1 END) AS not_in_metric,
  SUM(CASE WHEN metrics.interest_id is NULL THEN 1 END) AS not_in_map
FROM fresh_segments.interest_map map
FULL OUTER JOIN fresh_segments.interest_metrics metrics
  ON metrics.interest_id = map.id;
```

<kbd><img width="617" alt="image" src="https://github.com/user-attachments/assets/25311075-b607-4b77-975c-fd4311194f3c"></kbd>

- There are 1,209 unique `id`s in `interest_map`.
- There are 1,202 unique `interest_id`s in `interest_metrics`.
- There are no `interest_id` that did not appear in `interest_map`. All 1,202 ids were present in the `interest_metrics` table.
- There are 7 `id`s that did not appear in `interest_metrics`.
```sql
SELECT id AS Ids_not_in_metrics
FROM interest_map
WHERE id NOT IN (SELECT interest_id FROM interest_metrics);
```
<kbd><img width="120" alt="image" src="https://github.com/user-attachments/assets/d75793b6-c1e2-4055-9501-24a9092ced05"></kbd>
***
  
**5. Summarise the id values in the `fresh_segments.interest_map` by its total record count in this table.**

```sql
SELECT
  m.id,
  m.interest_name,
  COUNT(*) AS total_associations
FROM interest_map AS m
INNER JOIN interest_metrics AS me ON m.id = me.interest_id
GROUP BY m.id, m.interest_name
ORDER BY total_associations DESC;
```
Screen Shot :
<kbd>![image](https://github.com/user-attachments/assets/956dfe39-9971-4d8d-aa8a-61631401c4f9)</kbd>


***
  
**6. What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where 'interest_id = 21246' in your joined output and include all columns from `fresh_segments.interest_metrics` and all columns from `fresh_segments.interest_map` except from the id column.**

We should be using `INNER JOIN` to perform our analysis.

```sql
SELECT
  _month AS month,
  _year AS year,
  interest_id,
  composition,
  index_value,
  ranking,
  percentile_ranking,
  month_year
  interest_name,
  interest_summary,
  created_at,
  last_modified
FROM interest_metrics me
INNER JOIN interest_map m ON me.interest_id = m.id
WHERE interest_id = 21246
```

![image](https://github.com/user-attachments/assets/d0cc2dea-7326-42bb-af44-0b284b03c089)




The results should come up to 10 rows only. 

***

**7. Are there any records in your joined table where the `month_year` value is before the `created_at` value from the `fresh_segments.interest_map` table? Do you think these values are valid and why?**

```sql
SELECT 
  COUNT(*)
FROM fresh_segments.interest_map map
INNER JOIN fresh_segments.interest_metrics metrics
  ON map.id = metrics.interest_id
WHERE metrics.month_year < map.created_at::DATE;
```


<kbd><img width="106" alt="image" src="https://github.com/user-attachments/assets/ba3c2d68-33b5-45c5-91b2-538a53a1c05a"></kbd>

There are 188 records where the `month_year` date is before the `created_at` date. 

However, it looks like these records are created in the same month as `month_year`. Do you remember that the `month_year` column's date is set to default on 1st day of the month? 


<kbd><img width="761" alt="image" src="https://github.com/user-attachments/assets/12cbf9c6-c872-44b1-b3e0-817a1cc70cae"></kbd>

Running another test to see whether date in `month_year` and `created_at` are in the same month.

```sql
SELECT 
  COUNT(*)
FROM fresh_segments.interest_map map
INNER JOIN fresh_segments.interest_metrics metrics
  ON map.id = metrics.interest_id
WHERE metrics.month_year < DATE_TRUNC('mon', map.created_at::DATE);
```


<kbd><img width="110" alt="image" src="https://github.com/user-attachments/assets/4ad02f6e-a546-4e7b-8d56-e33aac15f797"></kbd>

Seems like all the records' dates are in the same month, hence we will consider the records as valid. 

***

## 📚 B. Interest Analysis
  
**1. Which interests have been present in all `month_year` dates in our dataset?**

Let's Find out how many unique `month_year` in dataset and the unique intrests are there 

```sql
SELECT 
  COUNT(DISTINCT month_year) AS unique_month_year_count, 
  COUNT(DISTINCT interest_id) AS unique_interest_id_count
FROM fresh_segments.interest_metrics;
```
<kbd><img width="465" alt="image" src="https://github.com/user-attachments/assets/b19ed11d-ad97-4faf-bc70-3a51149289cf"></kbd>


There are 14 distinct `month_year` dates and 1202 distinct `interest_id`s.

```sql

WITH interest_cte AS (
SELECT 
  interest_id, 
  COUNT(DISTINCT month_year) AS total_months_present
FROM fresh_segments.interest_metrics
WHERE month_year IS NOT NULL
GROUP BY interest_id
)

SELECT 
  c.total_months_present,
  COUNT(DISTINCT c.interest_id) AS all_months_present_interest_ids
FROM interest_cte c
WHERE total_months_present = 14
GROUP BY c.total_months_present
;

```
<kbd><img width="465" alt="image" src="https://github.com/user-attachments/assets/609144ef-8b53-48fa-8863-5220e7ad6226"></kbd>

480 interests out of 1202 interests are present in all the `month_year` dates.

***
  
**2. Using this same total_months measure - calculate the cumulative percentage of all records starting at 14 months - which total_months value passes the 90% cumulative percentage value?**

Find out the point in which interests present in a particular number of months are not performing well. For example, interest id 101 only appeared in 6 months due to non or lack of clicks and interactions, so we can consider to cut the interest off. 

```sql

WITH cte_interest_months AS (
SELECT
  interest_id,
  COUNT(DISTINCT month_year) AS total_months_present
FROM fresh_segments.interest_metrics
WHERE interest_id IS NOT NULL
GROUP BY interest_id
),
cte_interest_counts AS (
  SELECT
    total_months_present,
    COUNT(DISTINCT interest_id) AS interest_count
  FROM cte_interest_months
  GROUP BY total_months_present
)

SELECT
  total_months_present,
  interest_count,
  ROUND(100 * SUM(interest_count) OVER (ORDER BY total_months_present DESC) / -- Create running total field using cumulative values of interest count
      (SUM(INTEREST_COUNT) OVER ()),2) AS cumulative_percentage
FROM cte_interest_counts;
```

<kbd><img width="446" alt="image" src="https://github.com/user-attachments/assets/833282dc-e29c-4efd-8ffd-5c5971f5d21b"></kbd>

Interests with total months of 6 and below received a 90% and above percentage. Interests above this mark should be investigated to improve their clicks and customer interactions. 
***

**3. If we were to remove all `interest_id` values which are lower than the `total_months` value we found in the previous question - how many total data points would we be removing?**
```sql
WITH month_count AS
(		
SELECT interest_id, COUNT(DISTINCT (month_year)) AS month_count
FROM interest_metrics
GROUP BY interest_id
HAVING COUNT(DISTINCT month_year) < 6
)
		
--getting the number of times the above interest ids are present in the interest_metrics table
SELECT
    COUNT(interest_id) AS interest_record_to_remove
FROM
    interest_metrics
WHERE
    interest_id IN (
        SELECT
            interest_id
        FROM
            month_count
    );

```
<kbd>![image](https://github.com/user-attachments/assets/46e95dc2-37b4-4923-88da-9db8df5634fe)</kbd>


***

**4. Does this decision make sense to remove these data points from a business perspective? Use an example where there are all 14 months present to a removed interest example for your arguments - think about what it means to have less months present from a segment perspective. **

```sql
WITH month_counts AS (
    SELECT
        interest_id,
        COUNT(DISTINCT month_year) AS month_count
    FROM
        interest_metrics
    GROUP BY
        interest_id
    HAVING
        COUNT(DISTINCT month_year) < 6
)

SELECT
    removed.month_year,
    present_interest,
    removed_interest,
    ROUND(removed_interest * 100.0 / (removed_interest + present_interest), 2) AS removed_prcnt
FROM
    (
        SELECT
            month_year,
            COUNT(*) AS removed_interest
        FROM
            interest_metrics
        WHERE
            interest_id IN (SELECT interest_id FROM month_counts)
        GROUP BY
            month_year
    ) AS removed
JOIN
    (
        SELECT
            month_year,
            COUNT(*) AS present_interest
        FROM
            interest_metrics
        WHERE
            interest_id NOT IN (SELECT interest_id FROM month_counts)
        GROUP BY
            month_year
    ) AS present
ON
    removed.month_year = present.month_year
ORDER BY
    removed.month_year;
```
<kbd>![image](https://github.com/user-attachments/assets/17e3726d-e86d-494a-9788-8ada9e07e585)</kbd>
As removed percentage is not significant, we can removed the data points
***

**5. If we include all of our interests regardless of their counts - how many unique interests are there for each month?**
  
***

## 🧩 C. Segment Analysis
 
1. Using the complete dataset - which are the top 10 and bottom 10 interests which have the largest composition values in any month_year? Only use the maximum composition value for each interest but you must keep the corresponding month_year
2. Which 5 interests had the lowest average ranking value?
3. Which 5 interests had the largest standard deviation in their percentile_ranking value?
4. For the 5 interests found in the previous question - what was minimum and maximum percentile_ranking values for each interest and its corresponding year_month value? Can you describe what is happening for these 5 interests?
5. How would you describe our customers in this segment based off their composition and ranking values? What sort of products or services should we show to these customers and what should we avoid?

***

## 👆🏻 D. Index Analysis

The `index_value` is a measure which can be used to reverse calculate the average composition for Fresh Segments’ clients. Average composition can be calculated by dividing the composition column by the index_value column rounded to 2 decimal places.

1. What is the top 10 interests by the average composition for each month?
2. For all of these top 10 interests - which interest appears the most often?
3. What is the average of the average composition for the top 10 interests for each month?
4. What is the 3 month rolling average of the max average composition value from September 2018 to August 2019 and include the previous top ranking interests in the same output shown below.
5. Provide a possible reason why the max average composition might change from month to month? Could it signal something is not quite right with the overall business model for Fresh Segments?

***

Do give me a 🌟 if you like what you're reading. Thank you! 🙆🏻‍♀️
