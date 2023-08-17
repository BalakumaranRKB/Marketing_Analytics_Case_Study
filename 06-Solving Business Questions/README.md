# 6. Business Questions
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------  

## Overview
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------

The following questions are part of the final case study quiz - these are example questions the Marketing team might be interested in!

## 1. Which film title was the most recommended for all customers?

``` sql
with cte1 as (
select title from category_recommendations
UNION ALL
select title from actor_recommendations)
select title,COUNT(title) as cnt
from cte1
group by title
order by cnt desc
```

Output:

|title	           | cnt|
|------------------|----|
|DOGMA FAMILY	   | 126|
|JUGGLER HARDLY	   | 123|
|STORM HAPPINESS   | 111|
|HANDICAP BOONDOCK | 109|
|WIFE TURN	       | 106|

> The fim title which was recomended the most was 'DOGMA FAMILY'


## 2. How many customers were included in the email campaign?



```sql
select COUNT(*) from final_data_asset
```

Output:

|count |
|------|
|599   |

> The total count of the number of customers that were included in the email campaign were 599

## 3. Out of all the possible films - what percentage coverage do we have in our recommendations?

``` sql
with all_recommendations as (
select title from category_recommendations
UNION
select title from actor_recommendations),
total_count as (select COUNT(film_id) as total_cnt from film_counts)
SELECT (COUNT(title)::numeric / (SELECT total_cnt FROM total_count))*100 AS ratio
FROM all_recommendations
```

Output:

|ratio      |
|-----------|
|25.88726514|

> Out of all the possible films we have 25 percentage coverage our recommendations.

## 4. What is the most popular top category?

``` sql
select category_name,COUNT(category_name)
FROM first_category_insights
GROUP BY 1
ORDER BY 2 DESC
```

Output:

|category_name|	count|
|-------------|------|
|Sports	      |   67 |
|Action	      |   60 |
|Sci-Fi	      |   58 |
|Animation	  |   50 |
|Foreign	  |   43 |
|Drama	      |   38 |

> As we can see sports is the most popular category

## 5. What is the 4th most popular top category?

```sql
with cte1 as (
select category_name,COUNT(category_name),RANK() OVER(ORDER BY COUNT(category_name) DESC) AS RNK
FROM first_category_insights
GROUP BY 1
ORDER BY 2 DESC)
SELECT * FROM CTE1
WHERE RNK = 4
```

Output:
|category_name|	count  |rnk |
|-------------|------- |----|
|Animation	  |    50  | 4  |

> As we can see Animation is 4th most popular top category.

## 6. What is the average percentile ranking for each customer in their top category rounded to the nearest 2 decimal places?

``` sql
select ROUND(AVG(percentile)::NUMERIC,2) from first_category_insights
```

Output:

|average |
|--------|
|5.10    |

> The average percentile ranking is 5.10

## 7. What is the cumulative distribution of the top 5 percentile values for the top category from the first_category_insights table rounded to the nearest round percentage?

```sql
select percentile,COUNT(*),ROUND(100*CUME_DIST() OVER(ORDER BY percentile)) from first_category_insights
GROUP BY percentile
LIMIT 5
```

Output:

|percentile	|count	|round |
|-----------|-------|------|
|1	        | 159	|4     |
|2	        | 126	|9     |
|3	        | 41	|13    |
|4	        | 50	|17    |
|5	        | 14	|22    |

> The above output shows the the cumulative distribution of the top 5 percentile values for the top category from the first_category_insights table.

## 8. What is the median of the second category percentage of entire viewing history?

```sql
SELECT percentile_cont(0.5) WITHIN GROUP (ORDER BY total_percentage) AS median_value
FROM second_category_insights;
```

Output:

|median_value |
|-------------|
|13           |

> The median of the second category percentage of entire viewing history is 13.


## 9. What is the 80th percentile of films watched featuring each customer’s favourite actor?

```sql
select percentile_cont(0.8) WITHIN GROUP (ORDER BY rental_count) AS eightieth_percentile from top_actor_counts
```

Output:

|eightieth_percentile|
|--------------------|
|        5           |

> The 80th percentile of films watched featuring each customer’s favourite actor is 5.

## 10. What was the average number of films watched by each customer?

```sql
select ROUND(AVG(total_count)) from total_counts
```

Output:

|   avg |
|-------|
|27     |

> The average number of films watched by each customer is 27.

## 11.What is the top combination of top 2 categories and how many customers if the order is relevant (e.g. Horror and Drama is a different combination to Drama and Horror)

```sql
with cte2 as (
select tc1.customer_id,tc1.category_name as CAT1,tc2.category_name AS CAT2
from top_categories tc1
join top_categories tc2
on tc1.customer_id = tc2.customer_id
where tc1.category_rank =  1 and tc2.category_rank =  2)
select CAT1,CAT2 ,COUNT(*)
from cte2
group by CAT1,CAT2
order by COUNT(*) DESC
```

Output:

|cat1	   | cat2	    |  count|
|--------- |------------|-------|
|Sports	   | Animation	|   11  |
|Action	   | Documentary|	9   |
|Animation | Drama	    |   9   |
|Sci-Fi	   | Family	    |   8   |
|Action	   | Foreign	|   7   |
|Drama	   | Family	    |   7   |
|Animation | Family	    |   7   |
|Sports	   | Games	    |   7   |

> Sports and Animantion was the top combination if order were to be relevant.

# Q12. Which actor was the most popular for all customers?

```sql
with cte1 as (
select actor,COUNT(actor) as cnt,
RANK() OVER(ORDER BY COUNT(actor) DESC) AS rnk
from final_data_asset
GROUP BY actor)
select actor,cnt 
from cte1
where rnk = 1
```

Output:

|actor	          |cnt |
|-----------------|----|
|Walter Torn      |13  |
|Gina Degeneres   |13  |

> Walter Tom and Gina Degeneres are the most popular actors.



# 13. How many films on average had customers already seen that feature their favourite actor rounded to closest integer?

``` sql
select ROUND(AVG(rental_count)) as average from top_actor_counts
```

Output:

|average|
|-------|
|4      |

> There were 4 films on average which customers which customers had seen that feature their favourite actor.


## 14 What is the most common top categories combination if order was irrelevant and how many customers have this combination? (e.g. Horror and Drama is a the same as Drama and Horror)

```sql
SELECT
  LEAST(cat_1, cat_2) AS category_1,
  GREATEST(cat_1, cat_2) AS category_2,
  COUNT(*) AS freq_count
FROM final_data_asset
GROUP BY
  category_1,
  category_2
ORDER BY freq_count DESC
LIMIT 5;
```

Output:

|category_1	    |     category_2	 |   freq_count |
|---------------|--------------------|--------------|
|Animation	    |     Sports	     |       14     |
|Action	        |     Documentary	 |       12     |
|Family	        |     Sci-Fi	     |       12     |
|Animation	    |     Family	     |       12     |
|Documentary	|     Drama	         |       12     |

> If order was irrelevant the most popular combination was Animation and Sports.



 















