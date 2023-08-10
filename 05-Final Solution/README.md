# 5. Final Solution
--------------------------------------------------------------------------------------------------------------------------------------------------------------------

Having identified the key columns and completed our table analysis and required calculated column calculation, it's time to proceed with implementing the final solution. This will provide all the necessary answers required by the business team for sending the email template.


## 5.1 Category Insights
--------------------------------------------------------------------------------------------------------------------------------------------------------------------
Let us start by creating temporaray tables one by one and combine them into a single SQL script

### 5.1.1 Create Base Dataset

The base dataset is created by joining all tables except for the actor and actor_id table. The rental_date column is incorpotated to help us split ties for rankings which had the same count of rentals at a customer level.

``` SQL
DROP TABLE IF EXISTS complete_joint_dataset;
CREATE TEMP TABLE complete_joint_dataset AS
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title,
  category.name AS category_name,
  -- also included rental_date for sorting purposes
  rental.rental_date
FROM dvd_rentals.rental
INNER JOIN dvd_rentals.inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN dvd_rentals.film
  ON inventory.film_id = film.film_id
INNER JOIN dvd_rentals.film_category
  ON film.film_id = film_category.film_id
INNER JOIN dvd_rentals.category
  ON film_category.category_id = category.category_id;
```

```SQL
SELECT * FROM complete_joint_dataset limit 10;
```

Output:


|customer_id | film_id |  title	        | category_id    | category   |
|----------- | ------- |--------------- |----------------|------------|
|130	     |  80	   |BLANKET BEVERLY	|    8	         | Family     |
|459	     |  333	   |FREAKY POCUS	|    12	         | Music      |
|408	     |  373	   |GRADUATE LORD	|    3	         | Children   |
|333	     |  535	   |LOVE SUICIDES	|    11	         | Horror     |
|222	     |  450	   |IDOLS SNATCHERS	|    3	         | Children   |
|549	     |  613	   |MYSTIC TRUMAN	|    5	         |  Comedy    |
|269	     |  870	   |SWARM GOLD		|    11          |  Horror    |
|239	     |  510	   |LAWLESS VISION	|    2	         |  Animation |
|126	     |  565	   |MATRIX SNOWMAN	|    9	         |  Foreign   |
|399	     |  396	   |HANGING DEEP	|    7	         |  Drama     |


### 5.1.2 Category Counts

Now, that we have the total dataset table containing records for each customer's rental along with its category, let's calculate the category count for each customer's rental records. Also, lets take a look at the records when customer_id = 1.

```SQL
DROP TABLE IF EXISTS category_counts;
CREATE TEMP TABLE category_counts AS 
select customer_id,
       category_name,
       COUNT(*) as rental_count,
       MAX(rental_date) as latest_rental_date
       from complete_inner_join_dataset_with_rental_date
       GROUP BY 
                customer_id,
                category_name

SELECT *
FROM category_counts
WHERE customer_id = 1
ORDER BY
    rental_count DESC,
    latest_rental_date DESC;
```

Output:

|customer_id | category_name  |	 rental_count     |	   latest_rental_date      |
|------------|----------------|-------------------|----------------------------|
|1	         |   Classics	  |   6	              |    2005-08-19T09:55:16.000Z|
|1	         |   Comedy	      |   5	              |    2005-08-22T19:41:37.000Z|
|1	         |   Drama	      |   4	              |    2005-08-18T03:57:29.000Z|
|1	         |   Animation	  |   2	              |    2005-08-22T20:03:46.000Z|
|1	         |   Sci-Fi	      |   2	              |    2005-08-21T23:33:57.000Z|
|1	         |   New	      |   2	              |    2005-08-19T13:56:54.000Z|
|1	         |   Action	      |   2	              |    2005-08-17T12:37:54.000Z|
|1	         |   Music	      |   2	              |    2005-07-09T16:38:01.000Z|
|1	         |   Sports	      |   2	              |    2005-07-08T07:33:56.000Z|
|1	         |   Family	      |   1	              |    2005-08-02T18:01:38.000Z|
|1	         |   Documentary  |   1	              |    2005-08-01T08:51:04.000Z|
|1	         |   Foreign	  |   1	              |    2005-07-28T16:18:23.000Z|
|1	         |   Travel	      |   1	              |    2005-07-11T10:13:46.000Z|
|1	         |   Games	      |   1	              |    2005-07-08T03:17:05.000Z|

### 5.1.3 Total Counts
Finding the total number of films watched by each customer

```SQL
DROP TABLE IF EXISTS total_counts;
CREATE TEMP TABLE total_counts AS 
select customer_id,
       SUM(rental_count) as total_count
FROM category_counts
GROUP BY customer_id

SELECT *
FROM total_counts
LIMIT 5;
```

Output:
|customer_id  |	total_count  | 
|-------------|--------------|
|      184	  |  23          |
|      87	  |  30          |
|      477	  |  22          |
|      273	  |  35          |
|      550	  |  32          |

### 5.1.4. Top Categories
Using the DENSE_RANK window function to generate a ranking of categories for each customer.

```SQL
DROP TABLE IF EXISTS top_categories;
CREATE TEMP TABLE top_categories AS 
WITH ranked_cte as (
                     SELECT customer_id,
                            category_name,
                            rental_count,
                            DENSE_RANK() OVER(
                            PARTITION BY customer_id 
                            ORDER BY  
                                    rental_count desc,
                                    latest_rental_date desc,
                                    category_name) as category_rank
                      FROM category_counts
                                    
                    )
select * from ranked_cte
WHERE category_rank <=2
```

 Having a look at the output:
 ```SQL
 select * from top_categories LIMIT 5
 ```
 
 Output:
 
 |customer_id |category_name  |   rental_count|   category_rank   |
 |------------|-------------- |---------------|-------------------|
 |     1	  |     Classics  |      6	      |       1           |
 |     1	  |     Comedy	  |      5	      |       2           |
 |     2	  |     Sports	  |      5	      |       1           |
 |     2	  |     Classics  |      4	      |       2           |
 |     3	  |     Action	  |      4	      |       1           |
 
 
 ###  5.1.5 Top Category Percentile
Next we shall calculate the average category count for each category from the category counts table and use the FLOOR() function to round down to nearest decimal. We will also have a look at the first 5 rows. 

```SQL
DROP TABLE IF EXISTS top_category_percentile;
CREATE TEMP TABLE top_category_percentile AS
WITH calculated_cte as(
SELECT
  top_categories.customer_id,
  top_categories.category_name AS top_category_name,
  top_categories.rental_count, 
  category_counts.category_name,
  top_categories.category_rank,
  PERCENT_RANK() OVER (
    PARTITION BY category_counts.category_name
    ORDER BY category_counts.rental_count DESC
  ) AS raw_percentile_value
FROM category_counts
LEFT JOIN top_categories
  ON category_counts.customer_id = top_categories.customer_id 
)
SELECT 
      customer_id,
      category_name,
      rental_count,
      category_rank,
      CASE
           WHEN ROUND(100 * raw_percentile_value) = 0 THEN 1
           ELSE ROUND(100 * raw_percentile_value)
      END AS percentile
FROM calculated_cte
WHERE category_rank =  1
     AND top_category_name = category_name;

SELECT * FROM average_category_count LIMIT 5
```

Output:

|customer_id  |	category_name |	rental_count | 	category_rank |	percentile |
|-------------|---------------|--------------|----------------|------------|
|        1	  |   Classics	  |        6	 |         1	  |          1 |
|        2	  |   Sports	  |        5	 |         1	  |          2 |
|        3	  |   Action	  |        4	 |         1	  |          4 |
|        4	  |   Horror	  |        3	 |         1	  |          8 |
|        5	  |   Classics	  |        7	 |         1	  |          1 |


###  5.1.6 1st Category Insights

Combining the top_category_percentile table with the average category count table to generate our 1st Insights table:

```SQL 
DROP TABLE IF EXISTS first_category_insights;
CREATE TEMP TABLE first_category_insights AS 
SELECT 
      base.customer_id,
      base.category_name,
      base.rental_count,
      base.rental_count - average.category_average as average_comparsion,
      base.percentile
FROM top_category_percentile as base
left join average_category_count as average
  on base.category_name = average.category_name
  
select * from first_category_insights LIMIT 5
```

Output:

|customer_id | 	category_name |	rental_count | 	average_comparsion |  percentile |
|----------- |----------------|--------------|---------------------|-------------|
|     323	 |    Action	  |        7	 |         5	       |      1      |
|     506	 |    Action	  |        7	 |         5	       |      1      |
|     151	 |    Action	  |        6	 |         4	       |      1      |
|     410	 |    Action	  |        6	 |         4	       |      1      |
|     126	 |    Action	  |        6	 |         4	       |      1      |






###  5.1.7 2nd Category Insights

For the second top ranked category we calcualte the percentage of films watched for that category with respect to the total count of films watched by that customer.

```SQL
DROP TABLE IF EXISTS second_category_insights;
CREATE TEMP TABLE second_category_insights AS 
SELECT
       top_categories.customer_id,
       top_categories.category_name,
       top_categories.rental_count,
       ROUND(100 * top_categories.rental_count::NUMERIC/total_counts.total_count) as total_percentage
       from top_categories 
       left join total_counts 
       on top_categories.customer_id = total_counts.customer_id
       WHERE top_categories.category_rank = 2
```

Output:

|customer_id |	category_name |	rental_count |	total_percentage |
|------------|----------------|------------- |-------------------|
|  184	     |  Drama	      |   3	         |   13              |
|  87	     |  Sci-Fi	      |   3	         |   10              |
|  477	     |  Travel	      |   3	         |   14              |
|  273	     |  New	          |   4	         |   11              |
|  550	     |  Drama	      |   4	         |   13              |


## 5.2 Category Recomendations
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

### 5.2.1 Film Counts

We need to calculate the total number of films which were rented out and we would do this by using the base table which was created earlier.


```SQL
DROP TABLE IF EXISTS film_counts;
CREATE TEMP TABLE film_counts AS 
SELECT DISTINCT
               film_id,
               title,
               category_name,
               COUNT(*)  OVER(PARTITION BY film_id) as rental_count
FROM complete_inner_join_dataset_with_rental_date
```

Output:

|film_id  |	  title	          | category_name |	rental_count |
|---------|-------------------|---------------|------------- |
|655	  |  PANTHER REDS	  |    Sci-Fi	  |       15     |
|285	  |  ENGLISH BULWORTH |	   Sci-Fi	  |       30     |
|258	  |  DRUMS DYNAMITE	  |    Horror	  |       13     |
|809	  |  SLIPPER FIDELITY |	   Sports	  |       16     |
|883	  |  TEQUILA PAST	  |    Children	  |       6      |


### 5.2.2 Category Film Exclusions

We will keep a track of the films watched by each customer so that we won't be recommending films which they had watched already in the past.

```SQL
DROP TABLE IF EXISTS category_film_exclusions;
CREATE TEMP TABLE category_film_exclusions AS 
          SELECT DISTINCT customer_id,film_id
          FROM complete_inner_join_dataset_with_rental_date
		  
```
Having a look at the output:

Output:
|customer_id  |	film_id  |
|-------------|----------|
|      596	  |    103   |
|      176	  |    121   |
|      459	  |    724   |
|      375	  |    641   |
|      153	  |    730   |


### 5.2.2 Category Recommendations

We can finally recommend 3 films for each customer using the top categories table and film_counts table by joining them with an inner join. We then perform an anti-join with the category film exclusions table so that we recommend popular films by category which were not viewed by the customer.

We will then perform a window function to select the top 3 films for each of the top 2 categories per customer. To avoid random ties - we will sort by the title alphabetically in case the rental_count values are equal in the ORDER BY clause for our window function.

```SQL
DROP TABLE IF EXISTS category_recommendations;
CREATE TEMP TABLE category_recommendations AS
WITH ranked_films_cte AS (
  SELECT
    top_categories.customer_id,
    top_categories.category_name,
    top_categories.category_rank,
    -- why do we keep this `film_id` column you might ask?
    -- you will find out later on during the actor level recommendations!
    film_counts.film_id,
    film_counts.title,
    film_counts.rental_count,
    DENSE_RANK() OVER (
      PARTITION BY
        top_categories.customer_id,
        top_categories.category_rank
      ORDER BY
        film_counts.rental_count DESC,
        film_counts.title
    ) AS reco_rank
  FROM top_categories
  INNER JOIN film_counts
    ON top_categories.category_name = film_counts.category_name
  WHERE NOT EXISTS(  SELECT 1
                     FROM  category_film_exclusions
                     WHERE category_film_exclusions.customer_id = top_categories.customer_id AND 
                     category_film_exclusions.film_id = film_counts.film_id
  )
)
SELECT * FROM ranked_films_cte
WHERE reco_rank <= 3;
```

Having a look at the output for the first 2 categories for each customer(for the 1st 18 rows)

``` SQL
SELECT *
FROM category_recommendations
WHERE customer_id = 1
ORDER BY category_rank, reco_rank;
```

Output:

|customer_id |  category_name | category_rank |	film_id	|        title	         |rental_count	| reco_rank |
|------------|--------------- |---------------|---------|----------------------- |--------------|-----------|
|      1	 |    Classics	  |       1	      | 891	    |   TIMBERLAND SKY	     |    31	    |    1      |
|      1	 |    Classics	  |       1	      | 358	    |   GILMORE BOILED	     |    28	    |    2      |
|      1	 |    Classics	  |       1	      | 951	    |   VOYAGE LEGALLY	     |    28	    |    3      |
|      1	 |    Comedy	  |       2	      | 1000	|   ZORRO ARK	         |    31	    |    1      |
|      1	 |    Comedy	  |       2	      | 127	    |   CAT CONEHEADS	     |    30	    |    2      |
|      1	 |    Comedy	  |       2	      | 638	    |   OPERATION OPERATION	 |    27	    |    3      |


## 5.3 Actor Insights
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

### 5.3.1 Actor Joint Table

Here in this Actor Joint table we would need to reconstruct the base table as the last two tables i.e. dvd_rentals.film_actor and dvd_rental.actor have to be included. This new table will give us a list of customer rentals along with the actors that starred in it.

```SQL
DROP TABLE IF EXISTS actor_joint_dataset;
CREATE TEMP TABLE actor_joint_dataset AS 
SELECT  
      customer_id,
      rental_date,
      rental.rental_id,
      film.film_id,
      film.title,
      actor.actor_id,
      actor.first_name,
      actor.last_name
FROM dvd_rentals.rental
INNER JOIN dvd_rentals.Inventory 
    ON rental.inventory_id = inventory.inventory_id
INNER JOIN dvd_rentals.film
    ON Inventory.film_id = film.film_id
INNER JOIN dvd_rentals.film_actor
    ON film.film_id = film_actor.film_id
INNER JOIN dvd_rentals.actor
    ON film_actor.actor_id = actor.actor_id
```

 <details>
 <summary> Click to see sample rows and counts from actor_joint_dataset </summary>
 
           ```SQL
		   
                    SELECT
                      COUNT(*) AS total_row_count,
                      COUNT(DISTINCT rental_id) AS unique_rental_id,
                      COUNT(DISTINCT film_id) AS unique_film_id,
                      COUNT(DISTINCT actor_id) AS unique_actor_id,
                      COUNT(DISTINCT customer_id) AS unique_customer_id
                    FROM actor_joint_dataset;
			```
		Output:
		         |total_row_count |	unique_rental_id  |	unique_film_id	| unique_actor_id | unique_customer_id |
				 |--------------- |-------------------|-----------------|-----------------|--------------------|
                 |    87980	      |        16004	  |      955	    |      200	      |     599            |
				 
 </details>
 
  <details>
 <summary> Click to see the first 10 rows of the actor_joint_dataset </summary>
 
           ```SQL
		   
                SELECT *
						FROM actor_joint_dataset
						LIMIT 10;
			```
		Output:
		        |customer_id |	      rental_date	        | rental_id	| film_id |	       title	 |actor_id | first_name	 |last_name     |
				|------------|------------------------------|-----------|---------|----------------- |---------|-------------|--------------|
                |    130	 |   2005-05-24T22:53:30.000Z	|     1	    |   80	  |   BLANKET BEVERLY|	200	   |    THORA	 |    TEMPLE    |
                |    130	 |   2005-05-24T22:53:30.000Z	|     1	    |   80	  |   BLANKET BEVERLY|	193	   |    BURT	 |    TEMPLE    |
                |    130	 |   2005-05-24T22:53:30.000Z	|     1	    |   80	  |   BLANKET BEVERLY|	173	   |    ALAN	 |    DREYFUSS  |
                |    130	 |   2005-05-24T22:53:30.000Z	|     1	    |   80	  |   BLANKET BEVERLY|	16	   |    FRED	 |    COSTNER   |
                |    459	 |   2005-05-24T22:54:33.000Z	|     2	    |   333	  |   FREAKY POCUS	 |  147	   |    FAY	     |    WINSLET   |
                |    459	 |   2005-05-24T22:54:33.000Z	|     2	    |   333	  |   FREAKY POCUS	 |  127	   |    KEVIN	 |    GARLAND   |
                |    459	 |   2005-05-24T22:54:33.000Z	|     2	    |   333	  |   FREAKY POCUS	 |  105	   |    SIDNEY	 |    CROWE     |
                |    459	 |   2005-05-24T22:54:33.000Z	|     2	    |   333	  |   FREAKY POCUS	 |  103	   |    MATTHEW	 |    LEIGH     |
                |    459	 |   2005-05-24T22:54:33.000Z	|     2	    |   333	  |   FREAKY POCUS	 |  42	   |    TOM	     |    MIRANDA   |
                |    408	 |   2005-05-24T23:03:39.000Z	|     3	    |   373	  |   GRADUATE LORD	 |  140	   |    WHOOPI	 |    HURT      |

				 
 </details>
 

### 5.3.2 Top Actor Counts
Now, based on the actor_joint_dataset, we calculate the count of the total number of actors film a customer has watched and then select the top actor based on the DENSE_RANK() window function value. This will take care of our first actor insights requirement for the email template.

```SQL
DROP TABLE IF EXISTS top_actor_counts;
CREATE TEMP TABLE top_actor_counts AS
WITH actor_counts AS (
  SELECT
    customer_id,
    actor_id,
    first_name,
    last_name,
    COUNT(*) AS rental_count,
    -- we also generate the latest_rental_date just like our category insight
    MAX(rental_date) AS latest_rental_date
  FROM actor_joint_dataset
  GROUP BY
    customer_id,
    actor_id,
    first_name,
    last_name
),
ranked_actor_counts AS (
  SELECT
        actor_counts.*,
        DENSE_RANK() OVER(
        PARTITION BY customer_id
        ORDER BY 
                 rental_count DESC,
                 latest_rental_date DESC,
                 first_name,
                 last_name
        ) as actor_rank
  FROM actor_counts
  
  )
  SELECT
         customer_id,
         actor_id,
         first_name,
         last_name,
         rental_count
    FROM ranked_actor_counts
    where actor_rank = 1;
```
Having a look at the top 10 rows of the top_actor_counts table:

```SQL
select * from top_actor_counts LIMIT 10
```

Output:
  
  |customer_id |	actor_id |	first_name |  last_name   |	rental_count |
  |------------|------------ |-------------|--------------|------------- |
  |      1	   |    37	     |  VAL	       |  BOLGER	  |     6	     | 
  |      2	   |    107	     |  GINA	   |  DEGENERES	  |     5	     | 
  |      3	   |    150	     |  JAYNE	   |  NOLTE	      |     4	     | 
  |      4	   |    102	     |  WALTER	   |  TORN	      |     4	     | 
  |      5	   |    12	     |  KARL	   |  BERRY	      |     4	     | 
  |      6	   |    191	     |  GREGORY	   |  GOODING	  |     4	     | 
  |      7	   |    65	     |  ANGELA	   |  HUDSON	  |     5	     | 
  |      8	   |    167	     |  LAURENCE   | BULLOCK	  |     5	     | 
  |      9	   |    23	     |  SANDRA	   |  KILMER	  |     3	     | 
  |      10	   |    12	     |  KARL	   |  BERRY	      |     4	     |


## 5.3 Actor Recommendations
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### 5.3.1 Actor Film Counts

The base table created earlier had many rows introduced into it primarily because of the many to many mapping of film_id and actor_ids. So now we would do a split aggregation to get the rental count of each film and then join it back to the base table(actor_joint_table)

``` SQL
DROP TABLE IF EXISTS actor_film_counts;
CREATE TEMP TABLE actor_film_counts AS
WITH film_counts AS (
  SELECT
    film_id,
    COUNT(DISTINCT rental_id) AS rental_count
  FROM actor_joint_dataset
  GROUP BY film_id
)
SELECT DISTINCT
  actor_joint_dataset.film_id,
  actor_joint_dataset.actor_id,
  -- why do we keep the title here? can you figure out why?
  actor_joint_dataset.title,
  film_counts.rental_count
FROM actor_joint_dataset
LEFT JOIN film_counts
  ON actor_joint_dataset.film_id = film_counts.film_id;
```

Output:

|film_id |actor_id  |	    title	       |  rental_count|
|--------|----------|----------------------|--------------|
|    1	 | 1	    |   ACADEMY DINOSAUR   |      23      |
|    1	 | 10	    |   ACADEMY DINOSAUR   |      23      |
|    1	 | 20	    |   ACADEMY DINOSAUR   |      23      |
|    1	 | 30	    |   ACADEMY DINOSAUR   |      23      |
|    1	 | 40	    |   ACADEMY DINOSAUR   |      23      |
|    1	 | 53	    |   ACADEMY DINOSAUR   |      23      |
|    1	 | 108	    |   ACADEMY DINOSAUR   |      23      |
|    1	 | 162	    |   ACADEMY DINOSAUR   |      23      |
|    1	 | 188	    |   ACADEMY DINOSAUR   |      23      |
|    1	 | 198	    |   ACADEMY DINOSAUR   |      23      |
|    2	 | 19	    |   ACE GOLDFINGER	   |       7      |
|    2	 | 85	    |   ACE GOLDFINGER	   |       7      |
|    2	 | 90	    |   ACE GOLDFINGER	   |       7      |
|    2	 | 160	    |   ACE GOLDFINGER	   |       7      |
|    3	 | 2	    |   ADAPTATION HOLES   |      12      |
|    3	 | 19	    |   ADAPTATION HOLES   |      12      |
|    3	 | 24	    |   ADAPTATION HOLES   |      12      |
|    3	 | 64	    |   ADAPTATION HOLES   |      12      |
|    3	 | 123	    |   ADAPTATION HOLES   |      12      |
|    4	 | 41	    |   AFFAIR PREJUDICE   |      23      |





### 5.3.2. Actor Film Exclusions

To ensure non-repetitive film recommendations, we'll merge our original joint dataset with the category recommendations table using an anti-join statement. This way, we can track customer-viewed films and provide recommendations containing the most frequently starred actors, while avoiding any repeated recommendations.

```SQL
DROP TABLE IF EXISTS actor_film_exclusions;
CREATE TEMP TABLE actor_film_exclusions AS
(
  SELECT DISTINCT
    customer_id,
    film_id
  FROM complete_joint_dataset
)
-- we use a UNION to combine the previously watched and the recommended films!
UNION
(
  SELECT DISTINCT
    customer_id,
    film_id
  FROM category_recommendations
);

```

<details>
<summary> Click here to see the sample rows of actor_film_exclusions  </summary>

			```SQL
			        select * from actor_film_exclusions LIMIT 10
            ```
			
			Output:
			       |customer_id	|  film_id |
				   |------------|----------|
                   |     493	|   567    |
                   |     114	|   789    |
                   |     596	|   103    |
                   |     176	|   121    |
                   |     459	|   724    |
                   |     375	|   641    |
                   |     153	|   730    |
                   |     291	|   285    |
                   |     1	    |   480    |
                   |     144	|   93     |


</details> 


### 5.3.3. Final Actor Recommendations

We can now conclude our analysis phase with this final ANTI JOIN and DENSE_RANK query to perform the same operation as category insights previously - only this time we will use top_actor_counts, actor_film_counts and actor_film_exclusions tables for our analysis.

```SQL
DROP TABLE IF EXISTS actor_recommendations;
CREATE TEMP TABLE actor_recommendations AS
WITH ranked_actor_films_cte AS (
  SELECT
    top_actor_counts.customer_id,
    top_actor_counts.first_name,
    top_actor_counts.last_name,
    top_actor_counts.rental_count,
    actor_film_counts.title,
    actor_film_counts.film_id,
    actor_film_counts.actor_id,
    DENSE_RANK() OVER (
      PARTITION BY
        top_actor_counts.customer_id
      ORDER BY
        actor_film_counts.rental_count DESC,
        actor_film_counts.title
    ) AS reco_rank
  FROM top_actor_counts
  INNER JOIN actor_film_counts
    -- join on actor_id instead of category_name!
    ON top_actor_counts.actor_id = actor_film_counts.actor_id
  -- This is a tricky anti-join where we need to "join" on 2 different tables!
  WHERE NOT EXISTS (
    SELECT 1
    FROM actor_film_exclusions
    WHERE
      actor_film_exclusions.customer_id = top_actor_counts.customer_id AND
      actor_film_exclusions.film_id = actor_film_counts.film_id
  )
)
SELECT * FROM ranked_actor_films_cte
WHERE reco_rank <= 3;
```

<details>
<summary> Click here to see the sample rows of actor_recommendations  </summary>

			```sql
			SELECT * FROM actor_recommendations
		    ORDER BY customer_id, reco_rank
			LIMIT 15;
            ```
			
			Output:
			       |customer_id	 |   first_name	  |   last_name	  |rental_count	|    title	             |film_id| actor_id |reco_rank|
				   |-------------|----------------|---------------|-------------|----------------------- |-------|--------- |---------|
                   |        1	 |   VAL	      |      BOLGER	  |   6	        |  PRIMARY GLASS	     |   697 |   37	    |   1     |
                   |        1	 |   VAL	      |      BOLGER	  |   6	        |  ALASKA PHANTOM	     |   12	 |   37	    |   2     |
                   |        1	 |   VAL	      |      BOLGER	  |   6	        |  METROPOLIS COMA	     |   572 |   37	    |   3     |
                   |        2	 |   GINA	      |     DEGENERES |	  5	        |  GOODFELLAS SALUTE     |   369 |   107	|   1     |
                   |        2	 |   GINA	      |     DEGENERES |	  5	        |  WIFE TURN	         |   973 |   107	|   2     |
                   |        2	 |   GINA	      |     DEGENERES |	  5	        |  DOGMA FAMILY	         |   239 |   107	|   3     |
                   |        3	 |   JAYNE	      |     NOLTE	  |   4	        |  SWEETHEARTS SUSPECTS  |   873 |   150	|   1     |
                   |        3	 |   JAYNE	      |     NOLTE	  |   4	        |  DANCING FEVER	     |   206 |   150	|   2     |
                   |        3	 |   JAYNE	      |     NOLTE	  |   4	        |  INVASION CYCLONE	     |   468 |   150	|   3     |
                   |        4	 |   WALTER	      |     TORN	  |   4	        |  CURTAIN VIDEOTAPE	 |   200 |   102	|   1     |
                   |        4	 |   WALTER	      |     TORN	  |   4	        |  LIES TREATMENT	     |   521 |   102	|   2     |
                   |        4	 |   WALTER	      |     TORN	  |   4	        |  NIGHTMARE CHILL	     |   624 |   102	|   3     |
                   |        5	 |   KARL	      |     BERRY	  |   4	        |  VIRGINIAN PLUTO	     |   945 |   12	    |   1     |
                   |        5	 |   KARL	      |     BERRY	  |   4	        |  STAGECOACH ARMAGEDDON |   838 |   12	    |   2     |
                   |        5	 |   KARL	      |     BERRY	  |   4	        |  TELEMARK HEARTBREAKERS|   880 |   12	    |   3     |
				   
</details> 


## 5.4 Key Table Outputs
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Lets identify our main tables which will be used for fulfilling the busines requirements

### 5.4.1 Customer Level Insights

> 1. ```first_category_insights```

<details>
<summary> Click here to see the sample rows of actor_recommendations  </summary>
<br>

			```SELECT *
               FROM first_category_insights
               LIMIT 5;
            ```
			
			Output:
			           |customer_id | category_name  |	rental_count| average_comparsion |  percentile |
                       |----------- |----------------|--------------|--------------------|-------------|
                       |     323	|    Action	     |        7	    |         5	         |      1      |
                       |     506	|    Action	     |        7	    |         5	         |      1      |
                       |     151	|    Action	     |        6	    |         4	         |      1      |
                       |     410	|    Action	     |        6	    |         4	         |      1      |
                       |     126	|    Action	     |        6	    |         4	         |      1      |
			       
</details>


> 2. ```second_category_insights```

<details>
<summary> Click here to see the sample rows of actor_recommendations  </summary>
<br>

			```SQL
			   SELECT *
               FROM second_category_insights
               LIMIT 5;
            ```
			
			Output:
			
			|customer_id |	category_name |	rental_count |	total_percentage |
            |------------|----------------|------------- |-------------------|
            |  184	     |  Drama	      |   3	         |   13              |
            |  87	     |  Sci-Fi	      |   3	         |   10              |
            |  477	     |  Travel	      |   3	         |   14              |
            |  273	     |  New	          |   4	         |   11              |
            |  550	     |  Drama	      |   4	         |   13              |
			       
</details>


> 3. ```top_actor_counts```

<details>
<summary> Click here to see the sample rows of actor_recommendations  </summary>
<br>

			```SQL
			   SELECT *
               FROM top_actor_counts
               LIMIT 10;
            ```
			
			Output:
			  |customer_id |	actor_id |	first_name |  last_name   |	rental_count |
              |------------|------------ |-------------|--------------|------------- |
              |      1	   |    37	     |  VAL	       |  BOLGER	  |     6	     | 
              |      2	   |    107	     |  GINA	   |  DEGENERES	  |     5	     | 
              |      3	   |    150	     |  JAYNE	   |  NOLTE	      |     4	     | 
              |      4	   |    102	     |  WALTER	   |  TORN	      |     4	     | 
              |      5	   |    12	     |  KARL	   |  BERRY	      |     4	     | 
              |      6	   |    191	     |  GREGORY	   |  GOODING	  |     4	     | 
              |      7	   |    65	     |  ANGELA	   |  HUDSON	  |     5	     | 
              |      8	   |    167	     |  LAURENCE   | BULLOCK	  |     5	     | 
              |      9	   |    23	     |  SANDRA	   |  KILMER	  |     3	     | 
              |      10	   |    12	     |  KARL	   |  BERRY	      |     4	     |
			       
</details>

### 5.4.2 recommendations

> 1. ```category_recommendations```

<details>
<summary> Click here to see the sample rows of actor_recommendations  </summary>
<br>

			```SQL
			   SELECT *
               FROM category_recommendations
               WHERE customer_id = 1
               ORDER BY category_rank, reco_rank
			   LIMIT 6;
            ```
			
			Output:
			
			          |customer_id |  category_name | category_rank |	film_id	|        title	         |rental_count	| reco_rank |
                      |------------|--------------- |---------------|-----------|----------------------- |--------------|-----------|
                      |      1	   |    Classics	|       1	    | 891	    |   TIMBERLAND SKY	     |    31	    |    1      |
                      |      1	   |    Classics	|       1	    | 358	    |   GILMORE BOILED	     |    28	    |    2      |
                      |      1	   |    Classics	|       1	    | 951	    |   VOYAGE LEGALLY	     |    28	    |    3      |
                      |      1	   |    Comedy	    |       2	    | 1000	    |   ZORRO ARK	         |    31	    |    1      |
                      |      1	   |    Comedy	    |       2	    | 127	    |   CAT CONEHEADS	     |    30	    |    2      |
                      |      1	   |    Comedy	    |       2	    | 638	    |   OPERATION OPERATION	 |    27	    |    3      |

			       
</details>


> 2. ```actor_recommendations```

<details>
<summary> Click here to see the sample rows of actor_recommendations  </summary>
<br>

			```SELECT *
               FROM actor_recommendations
               ORDER BY customer_id, reco_rank
               LIMIT 15;
            ```
			
			Output:
			
				   |customer_id	 |   first_name	  |   last_name	  |rental_count	|    title	             |film_id| actor_id |reco_rank|
				   |-------------|----------------|---------------|-------------|----------------------- |-------|--------- |---------|
                   |        1	 |   VAL	      |      BOLGER	  |   6	        |  PRIMARY GLASS	     |   697 |   37	    |   1     |
                   |        1	 |   VAL	      |      BOLGER	  |   6	        |  ALASKA PHANTOM	     |   12	 |   37	    |   2     |
                   |        1	 |   VAL	      |      BOLGER	  |   6	        |  METROPOLIS COMA	     |   572 |   37	    |   3     |
                   |        2	 |   GINA	      |     DEGENERES |	  5	        |  GOODFELLAS SALUTE     |   369 |   107	|   1     |
                   |        2	 |   GINA	      |     DEGENERES |	  5	        |  WIFE TURN	         |   973 |   107	|   2     |
                   |        2	 |   GINA	      |     DEGENERES |	  5	        |  DOGMA FAMILY	         |   239 |   107	|   3     |
                   |        3	 |   JAYNE	      |     NOLTE	  |   4	        |  SWEETHEARTS SUSPECTS  |   873 |   150	|   1     |
                   |        3	 |   JAYNE	      |     NOLTE	  |   4	        |  DANCING FEVER	     |   206 |   150	|   2     |
                   |        3	 |   JAYNE	      |     NOLTE	  |   4	        |  INVASION CYCLONE	     |   468 |   150	|   3     |
                   |        4	 |   WALTER	      |     TORN	  |   4	        |  CURTAIN VIDEOTAPE	 |   200 |   102	|   1     |
                   |        4	 |   WALTER	      |     TORN	  |   4	        |  LIES TREATMENT	     |   521 |   102	|   2     |
                   |        4	 |   WALTER	      |     TORN	  |   4	        |  NIGHTMARE CHILL	     |   624 |   102	|   3     |
                   |        5	 |   KARL	      |     BERRY	  |   4	        |  VIRGINIAN PLUTO	     |   945 |   12	    |   1     |
                   |        5	 |   KARL	      |     BERRY	  |   4	        |  STAGECOACH ARMAGEDDON |   838 |   12	    |   2     |
                   |        5	 |   KARL	      |     BERRY	  |   4	        |  TELEMARK HEARTBREAKERS|   880 |   12	    |   3     |
				   
			       
</details>

# 5.5 Final Output TABLE

Finally, lets create the output table that the marketing business table can use directly to fullfil the business requirements of the email marketing template.

```SQL
DROP TABLE IF EXISTS final_data_asset;
CREATE TEMP TABLE final_data_asset AS
WITH first_category AS (
  SELECT
    customer_id,
    category_name,
    CONCAT(
      'You''ve watched ', rental_count, ' ', category_name,
      ' films, that''s ', average_comparsion,
      ' more than the DVD Rental Co average and puts you in the top ',
      percentile, '% of ', category_name, ' gurus!'
    ) AS insight
  FROM first_category_insights
),
second_category AS (
  SELECT
    customer_id,
    category_name,
    CONCAT(
      'You''ve watched ', rental_count, ' ', category_name,
      ' films making up ', total_percentage,
      '% of your entire viewing history!'
    ) AS insight
  FROM second_category_insights
),
top_actor AS (
  SELECT
    customer_id,
    -- use INITCAP to transform names into Title case
    CONCAT(INITCAP(first_name), ' ', INITCAP(last_name)) AS actor_name,
    CONCAT(
      'You''ve watched ', rental_count, ' films featuring ',
      INITCAP(first_name), ' ', INITCAP(last_name),
      '! Here are some other films ', INITCAP(first_name),
      ' stars in that might interest you!'
    ) AS insight
  FROM top_actor_counts
),
adjusted_title_case_category_recommendations AS (
  SELECT
    customer_id,
    INITCAP(title) AS title,
    category_rank,
    reco_rank
  FROM category_recommendations
),
wide_category_recommendations AS (
  SELECT
    customer_id,
    MAX(CASE WHEN category_rank = 1  AND reco_rank = 1
      THEN title END) AS cat_1_reco_1,
    MAX(CASE WHEN category_rank = 1  AND reco_rank = 2
      THEN title END) AS cat_1_reco_2,
    MAX(CASE WHEN category_rank = 1  AND reco_rank = 3
      THEN title END) AS cat_1_reco_3,
    MAX(CASE WHEN category_rank = 2  AND reco_rank = 1
      THEN title END) AS cat_2_reco_1,
    MAX(CASE WHEN category_rank = 2  AND reco_rank = 2
      THEN title END) AS cat_2_reco_2,
    MAX(CASE WHEN category_rank = 2  AND reco_rank = 3
      THEN title END) AS cat_2_reco_3
  FROM adjusted_title_case_category_recommendations
  GROUP BY customer_id
),
adjusted_title_case_actor_recommendations AS (
  SELECT
    customer_id,
    INITCAP(title) AS title,
    reco_rank
  FROM actor_recommendations
),
wide_actor_recommendations AS (
  SELECT
    customer_id,
    MAX(CASE WHEN reco_rank = 1 THEN title END) AS actor_reco_1,
    MAX(CASE WHEN reco_rank = 2 THEN title END) AS actor_reco_2,
    MAX(CASE WHEN reco_rank = 3 THEN title END) AS actor_reco_3
  FROM adjusted_title_case_actor_recommendations
  GROUP BY customer_id
),
final_output AS (
  SELECT
    t1.customer_id,
    t1.category_name AS cat_1,
    t4.cat_1_reco_1,
    t4.cat_1_reco_2,
    t4.cat_1_reco_3,
    t2.category_name AS cat_2,
    t4.cat_2_reco_1,
    t4.cat_2_reco_2,
    t4.cat_2_reco_3,
    t3.actor_name AS actor,
    t5.actor_reco_1,
    t5.actor_reco_2,
    t5.actor_reco_3,
    t1.insight AS insight_cat_1,
    t2.insight AS insight_cat_2,
    t3.insight AS insight_actor
FROM first_category AS t1
INNER JOIN second_category AS t2
  ON t1.customer_id = t2.customer_id
INNER JOIN top_actor t3
  ON t1.customer_id = t3.customer_id
INNER JOIN wide_category_recommendations AS t4
  ON t1.customer_id = t4.customer_id
INNER JOIN wide_actor_recommendations AS t5
  ON t1.customer_id = t5.customer_id
)
SELECT * FROM final_output;
```
Having a look at the first 5 rows of this final_output table: 

``` SQL
SELECT * FROM final_data_asset
LIMIT 5;
```

Output:

|customer_id  |	  cat_1	  |    cat_1_reco_1	       |  cat_1_reco_2	    |   cat_1_reco_3	  |  cat_2	   | cat_2_reco_1	  |   cat_2_reco_2	 |  cat_2_reco_3	   |      actor	       |  actor_reco_1	       | actor_reco_2	      |      actor_reco_3   	  |                                                 insight_cat_1	                                                                 |                                               insight_cat_2	                    |                                                 insight_actor                                                     |
|-------------|-----------|------------------------|--------------------|---------------------|------------|------------------|------------------|---------------------|-------------------|-----------------------|----------------------|---------------------------|----------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------|
|      1	  |   Classics|    Timberland Sky	   |  Gilmore Boiled	|   Voyage Legally	  |  Comedy	   | Zorro Ark	      |   Cat Coneheads	 | Operation Operation |    Val Bolger	   |  Primary Glass	       | Alaska Phantom	      |    Metropolis Coma	      |  You've watched 6 Classics films, that's 4 more than the DVD Rental Co average and puts you in the top 1% of Classics gurus!	 |   You've watched 5 Comedy films making up 16% of your entire viewing history!    |  You've watched 6 films featuring Val Bolger! Here are some other films Val stars in that might interest you!     |
|      2	  |   Sports  |    Gleaming Jawbreaker |  Talented Homicide	|   Roses Treasure	  |  Classics  | Frost Head	      |   Gilmore Boiled |   Voyage Legally	   |    Gina Degeneres |  Goodfellas Salute	   | Wife Turn	          |    Dogma Family	          |  You've watched 5 Sports films, that's 3 more than the DVD Rental Co average and puts you in the top 2% of Sports gurus!	     |   You've watched 4 Classics films making up 15% of your entire viewing history!  |  You've watched 5 films featuring Gina Degeneres! Here are some other films Gina stars in that might interest you!| 
|      3	  |   Action  |    Rugrats Shakespeare |  Suspects Quills	|   Handicap Boondock |	 Sci-Fi	   | Goodfellas Salute|	 English Bulworth| Graffiti Love	   |    Jayne Nolte	   |  Sweethearts Suspects | Dancing Fever	      |    Invasion Cyclone	      |  You've watched 4 Action films, that's 2 more than the DVD Rental Co average and puts you in the top 4% of Action gurus!	     |   You've watched 3 Sci-Fi films making up 12% of your entire viewing history!    |  You've watched 4 films featuring Jayne Nolte! Here are some other films Jayne stars in that might interest you!  |
|      4	  |   Horror  |    Pulp Beverly	       |  Family Sweet	    |   Swarm Gold	      |  Drama	   | Hobbit Alien     |   Harry Idaho	 | Witches Panic	   |    Walter Torn	   |  Curtain Videotape	   | Lies Treatment	      |    Nightmare Chill	      |  You've watched 3 Horror films, that's 2 more than the DVD Rental Co average and puts you in the top 8% of Horror gurus!	     |   You've watched 2 Drama films making up 9% of your entire viewing history!	    |  You've watched 4 films featuring Walter Torn! Here are some other films Walter stars in that might interest you! |
|      5	  |   Classics|    Timberland Sky	   |  Frost Head	    |   Gilmore Boiled	  |  Animation | Juggler Hardly	  |   Dogma Family	 | Storm Happiness	   |    Karl Berry	   |  Virginian Pluto	   | Stagecoach Armageddon|    Telemark Heartbreakers |  You've watched 7 Classics films, that's 5 more than the DVD Rental Co average and puts you in the top 1% of Classics gurus!	 |   You've watched 6 Animation films making up 16% of your entire viewing history! |  You've watched 4 films featuring Karl Berry! Here are some other films Karl stars in that might interest you!    |




This output would be more than enough for the marketing team to run their email marketing campaign and hopefully attract customers.






 
 
																		
	





											 




 

 