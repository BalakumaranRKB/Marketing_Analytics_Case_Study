# 4 Problem Solving
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------

##   4.1 Revist of Base Table
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------
``` sql
DROP TABLE IF EXISTS complete_joint_dataset;
CREATE TEMP TABLE complete_joint_dataset AS
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title,
  rental.rental_date,
  category.name AS category_name
FROM dvd_rentals.rental
INNER JOIN dvd_rentals.inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN dvd_rentals.film
  ON inventory.film_id = film.film_id
INNER JOIN dvd_rentals.film_category
  ON film.film_id = film_category.film_id
INNER JOIN dvd_rentals.category
  ON film_category.category_id = category.category_id;

SELECT * FROM complete_joint_dataset limit 10;

```

Output:

|customer_id | film_id |  title	        | category_id	| category   |
|----------- | ------- |--------------- |-------------- |---------   |
|130	     |  80	   |BLANKET BEVERLY	|    8	        | Family     |
|459	     |  333	   |FREAKY POCUS	|        12	    | Music      |
|408	     |  373	   |GRADUATE LORD	|        3	    | Children   |
|333	     |  535	   |LOVE SUICIDES	|        11	    | Horror     |
|222	     |  450	   |IDOLS SNATCHERS	|    3	        | Children   |
|549	     |  613	   |MYSTIC TRUMAN	|        5	    |  Comedy    |
|269	     |  870	   |SWARM GOLD		|        11     |  Horror    |
|239	     |  510	   |LAWLESS VISION	|        2	    |  Animation |
|126	     |  565	   |MATRIX SNOWMAN	|        9	    |  Foreign   |
|399	     |  396	   |HANGING DEEP	|        7	    |  Drama     |


##   4.2 Rental Count of Customer(Category Level)
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------
To generate the names of the top 2 ranking categories of films watched by each customer, we would need to calculate the rental count, i.e., the total number of films they have watched for each category level. The same applies to other calculated fields, such as percentile and average comparison, where we need to calculate the number of films watched for each category, rather than limiting category counts to only the top 2 ranked categories

By taking an smaller subset from the previous base table where we take customer_ids of values 1,2,3.
We now generate each customers’ aggregated rental_count values for every category_name value from our complete_joint_dataset temporary table also.

Let’s also sort the output by customer_id and show the rental_count from largest to smallest for each customer:

```sql
SELECT
  customer_id,
  category_name,
  COUNT(*) AS rental_count,
  MAX(rental_date) AS latest_rental_date
FROM complete_joint_dataset
WHERE customer_id in (1, 2, 3)
GROUP BY
  customer_id,
  category_name
ORDER BY
  customer_id,
  rental_count DESC;
```

Spitting the output for each customer for easier viewing:

Customer 1
<details>
<summary> Click to view output </summary>

|customer_id|  category	    |rental_count |
|-----------|---------------|-------------|
|1	        |  Classics	    |     6       |
|1	        |  Comedy	    |     5       |
|1	        |  Drama	    |     4       |
|1	        |  Action	    |     2       |
|1	        |  Music	    |     2       |
|1	        |  New	        |     2       |
|1	        |  Sci-Fi	    |     2       |
|1	        |  Sports	    |     2       |
|1	        |  Animation	|     2       |
|1	        |  Documentary  |     1       |
|1	        |  Family	    |     1       |
|1	        |  Games	    |     1       |
|1	        |  Travel	    |     1       |
|1	        |  Foreign	    |     1       |
		
 </details>

Customer 2
<details>
<summary> Click to view output </summary>
 
|customer_id|  category	    |rental_count |
|-----------|---------------|-------------|
|  2	    |  Sports	    |   5         |
|  2	    |  Classics	    |   4         |
|  2	    |  Animation    |   3         |
|  2	    |  Action	    |   3         |
|  2	    |  Travel	    |   2         |
|  2	    |  Games	    |   2         |
|  2	    |  New	        |   2         |
|  2	    |  Foreign	    |   1         |
|  2	    |  Children	    |   1         |
|  2	    |  Documentary	|   1         |
|  2	    |  Family	    |   1         |
|  2	    |  Music	    |   1         |
|  2	    |  Sci-Fi	    |   1         |
		
</details>


Customer 3
<details>
<summary> Click to view output </summary>
 

|customer_id  |  category	|rental_count |
|-------------|-------------|-------------|   
|    3	    |    Action	    |     4       |
|    3	    |    Animation	|     3       |
|    3	    |    Sci-Fi	    |     3       |
|    3	    |    Sports	    |     2       |
|    3	    |    Comedy	    |     2       |
|    3	    |    Games	    |     2       |
|    3	    |    Horror	    |     2       |
|    3	    |    Music	    |     2       |
|    3	    |    New	    |     2       |
|    3	    |    Drama	    |     1       |
|    3	    |    Family	    |     1       |
|    3	    |    Documentary|     1       |
|    3	    |    Classics	|     1       |

 </details>

##   4.3 Dealing with ties


We want the top 2 categories for each customer based on the rental count but what if there is a tie in the rental count. We could sort the category alphabetically but it might not be the best solution.

Customer 3 categories sorted by rental count descending and alphabetical order

|customer_id  |	category_name |	rental_count |
|-------------| ------------- | ------------ |
|3	          |  Action	      |      4       |
|3	          | Animation	  |      3       |
|3	          |  Sci-Fi	      |      3       |


In this scenario, a tie occurs between the Animation and Sci-Fi film categories, each having a rental_count of 3. One simple and efficient approach is to sort the categories alphabetically and choose the 1st or 2nd category. This method is quick and easy to implement.

Alternatively, we can explore the rental_date column to determine which category the customer most recently purchased. By considering the latest rental_date, we can make a more informed selection. Let's now delve into the SQL implementation for customer 3.

    ```sql
    --firstly bring in rental_date as a field from the table joins
    DROP TABLE IF EXISTS complete_joint_dataset_with_rental_date;
    CREATE TEMP TABLE complete_joint_dataset_with_rental_date AS
    SELECT
      rental.customer_id,
      inventory.film_id,
      film.title,
      category.name AS category_name,
      rental.rental_date
    FROM dvd_rentals.rental
    INNER JOIN dvd_rentals.inventory
      ON rental.inventory_id = inventory.inventory_id
    INNER JOIN dvd_rentals.film
      ON inventory.film_id = film.film_id
    INNER JOIN dvd_rentals.film_category
      ON film.film_id = film_category.film_id
    INNER JOIN dvd_rentals.category
      ON film_category.category_id = category.category_id
    
    -- Finally perform group by aggregations on category_name and customer_id
    SELECT
      customer_id,
      category_name,
      COUNT(*) AS rental_count,
      MAX(rental_date) AS latest_rental_date
    FROM complete_joint_dataset_with_rental_date
    -- note the different filter here!
    WHERE customer_id = 3
    GROUP BY
      customer_id,
      category_name
    ORDER BY
      customer_id,
      rental_count DESC,
      latest_rental_date DESC;
    ```
Output:
	
|customer_id|	category_name	|rental_count |lastest_rental_date       |
|-----------|-------------------|------------ |--------------------------|                                                                 
|	3	    |   Action	        |4	          | 2005-07-29T11:07:04.000Z |
|	3	    |   Sci-Fi		    |3            | 2005-08-22T09:37:27.000Z |
|	3	    |   Animation		|3            | 2005-08-18T14:49:55.000Z |
|	3	    |   Music	        |2	          | 2005-08-23T07:10:14.000Z |
|	3	    |   Comedy	        |2	          | 2005-08-20T06:14:12.000Z |
|	3	    |   Horror	        |2	          | 2005-07-31T11:32:58.000Z |
|	3	    |   Sports	        |2	          | 2005-07-30T13:31:20.000Z |
|	3	    |   New		        |2            | 2005-07-28T04:46:30.000Z |
|	3	    |   Games	        |2	          | 2005-07-27T04:54:42.000Z |
|	3	    |   Classics	    |1	          | 2005-08-01T14:19:48.000Z |
|	3	    |   Family	        |1	          | 2005-07-31T03:27:58.000Z |
|	3	    |   Drama	        |1	          | 2005-07-30T21:45:46.000Z |
|	3	    |   Documentary		|1            | 2005-06-19T08:34:53.000Z |
	
Great - now we can see that Customer 3 most recent rental was from the Sci-Fi category. Earlier it was the Animation category.
	
##   4.3 Data Aggregation on Whole Dataset
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
We will aggregate the rental count values for each of the customer and category values and all these aggregations will be performed on the complete_joint_dataset temporary table that was created earlier.

The various aggregations and calculations are split up into various temporary tables. 


###   4.3.1 Customer Rental Count
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Let's first aggregate the rental_count for each customer's record for each category. Here, we'll also select the latest_rental_date for that category which will be useful for us in sorting as seen earlier.


``` sql
DROP TABLE IF EXISTS category_rental_counts;
CREATE TEMP TABLE category_rental_counts AS 
SELECT customer_id,
       category_name,
	   COUNT(*) AS rental_count,
	   MAX(rental_date) as latest_rental_date
FROM complete_inner_join_dataset_with_rental_date
GROUP BY customer_id,category_name

SELECT *
FROM category_rental_counts
WHERE customer_id = 1
ORDER BY
  rental_count DESC;
```

Output:

|customer_id|category_name  |rental_count	|       latest_rental_date      |
|-----------|---------------|---------------|-------------------------------|
|1	        |  Classics	    |     6	        |      2005-08-19T09:55:16.000Z |
|1	        |  Comedy	    |     5	        |      2005-08-22T19:41:37.000Z |
|1	        |  Drama	    |     4	        |      2005-08-18T03:57:29.000Z |
|1	        |  Sci-Fi	    |     2	        |      2005-08-21T23:33:57.000Z |
|1	        |  Animation	|     2	        |      2005-08-22T20:03:46.000Z |
|1	        |  Sports	    |     2	        |      2005-07-08T07:33:56.000Z |
|1	        |  Music		|     2         |      2005-07-09T16:38:01.000Z |
|1	        |  Action	    |     2	        |      2005-08-17T12:37:54.000Z |
|1	        |  New		    |     2         |      2005-08-19T13:56:54.000Z |
|1	        |  Travel	    |     1	        |      2005-07-11T10:13:46.000Z |
|1	        |  Family	    |     1	        |      2005-08-02T18:01:38.000Z |
|1	        |  Documentary	|     1	        |      2005-08-01T08:51:04.000Z |
|1	        |  Games		|     1         |      2005-07-08T03:17:05.000Z |
|1	        |  Foreign	    |     1	        |      2005-07-28T16:18:23.000Z |



###   4.3.2 Total customer rentals
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

In order to generate the category_percentage calculation we will need to get the total rentals per customer.

``` sql
DROP TABLE IF EXISTS customer_total_rentals;
CREATE TEMP TABLE customer_total_rentals AS 
SELECT customer_id,SUM(rental_count) as total_rental_count
from  category_rental_counts
group by customer_id


SELECT *
FROM customer_total_rentals
WHERE customer_id <= 5
ORDER BY customer_id;
```

Output:

|customer_id  |	total_rental_count|
|-------------| ------------------|
|       1	  |     32            |
|       2	  |     27            |
|       3	  |     26            |
|       4	  |     22            |
|       5	  |     38            |


###   4.3.3 Average category rental counts
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Using the AVG function to find the AVG category records for all customers which gives us the true average rental count for each category.

``` sql
DROP TABLE IF EXISTS avg_category_rental_counts;
CREATE TEMP TABLE avg_category_rental_counts AS 
select category_name,
       AVG(rental_count) as avg_rental_count 
       from category_rental_counts
       group by category_name
	   
select * from avg_category_rental_counts
order by avg_rental_count  desc
```

Output:

|category_name|  avg_rental_count       |
|-------------|-------------------------|
|Animation	  |  2.3320000000000000     |
|Sports	      |  2.2716763005780347     |
|Family	      |  2.1876247504990020     |
|Action	      |  2.1803921568627451     |
|Documentary  |  2.1739130434782609     |
|Sci-Fi	      |  2.1715976331360947     |
|Drama	      |  2.1157684630738523     |
|Foreign	  |  2.0953346855983773     |
|Games	      |  2.0443037974683544     |
|New	      |  2.0085470085470085     |
|Classics	  |  2.0064102564102564     |
|Children	  |  1.9605809128630705     |
|Comedy	      |  1.9010101010101010     |
|Travel	      |  1.8936651583710407     |
|Horror	      |  1.8758314855875831     |
|Music	      |  1.8568232662192394     |


To give our customers a feel-good boost about watching more films, instead of telling them they watched 1.346 more films than the average customer, we will take the FLOOR value of the decimal. This can be achieved by running an UPDATE command on the temp table we created.

```sql
UPDATE average_category_rental_counts
SET avg_rental_count = FLOOR(avg_rental_count)
RETURNING *;
```

Output:

|category_name |avg_rental_count   |
|------------- |-------------------|
|Sports	       |       2           |
|Classics	   |       2           |
|New	       |       2           |
|Family	       |       2           |
|Comedy	       |       1           |
|Animation	   |       2           |
|Travel	       |       1           |
|Music	       |       1           |
|Horror	       |       1           |
|Drama	       |       2           |
|Sci-Fi	       |       2           |
|Games	       |       2           |
|Documentary   |       2           |
|Foreign	   |       2           |
|Action	       |       2           |
|Children	   |       1           |


###   4.3.4 Percentile Rank
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
We can find the percentile values(customer ranking in terms of the top X% compared to all other customers in a given film category) using the PERCENT_RANK window function. Here is the below query:

``` sql
SELECT
  customer_id,
  category_name,
  rental_count,
  PERCENT_RANK() OVER (
    PARTITION BY category_name
    ORDER BY rental_count DESC
  ) AS percentile
FROM category_rental_counts
ORDER BY customer_id, rental_count DESC
LIMIT 14;
```

Output:

|customer_id| category_name |rental_count |	percentile  |
|-----------|---------------|-------------|-------------|
|1	        |  Classics	    |   6	      |  0.002141328|
|1	        |  Comedy	    |   5	      |  0.006072874|
|1	        |  Drama	    |   4	      |  0.03       |
|1	        |  Animation	|   2	      |  0.388777555|
|1	        |  New	        |   2	      |  0.267665953|
|1	        |  Action	    |   2	      |  0.333988212|
|1	        |  Music	    |   2	      |  0.204035874|
|1	        |  Sports	    |   2	      |  0.345559846|
|1	        |  Sci-Fi	    |   2	      |  0.300395257|
|1	        |  Documentary	|   1	      |  0.643153527|
|1	        |  Games	    |   1	      |  0.600422833|
|1	        |  Foreign	    |   1	      |  0.617886179|
|1	        |  Family	    |   1	      |  0.654      |
|1	        |  Travel	    |   1	      |  0.575963719|

The percent_rank() function gives us values between 0 to 1 but we would need to multiply by 100 and apply the ceiling function as rounding would create confusion if someone is interpreting the data.

``` sql
DROP TABLE IF EXISTS customer_category_percentiles;
CREATE TEMP TABLE customer_category_percentiles AS
SELECT
  customer_id,
  category_name,
  -- use ceiling to round up to nearest integer after multiplying by 100
  CEILING(
    100 * PERCENT_RANK() OVER (
      PARTITION BY category_name
      ORDER BY rental_count DESC
    )
  ) AS percentile
FROM category_rental_counts;

-- inspect top 2 records for customer_id = 1 sorted by ascending percentile
SELECT *
FROM customer_category_percentiles
WHERE customer_id = 1
ORDER BY customer_id, percentile
LIMIT 2;
```

Output:

|customer_id| category_name	|  rental_count	 |   percentile |
|-----------|---------------|----------------|--------------|
|1	        |     Classics	|         6	     |       1      |
|1	        |     Comedy	|         5	     |       1      |

After this transformation our percentile values which were earlier from 0 to 1 are now from 0 to 100.


###   4.3.5 Joining temporary tables
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
With all our aggregated values stored in temporary tables , we would now need to join them together and store it in a single temporary table. The following SQL code is shown below:

``` sql
DROP TABLE IF EXISTS customer_category_joint_table;
CREATE TEMP TABLE customer_category_joint_table AS
SELECT
  t1.customer_id,
  t1.category_name,
  t1.rental_count,
  t2.total_rental_count,
  t3.avg_rental_count,
  t4.percentile
FROM category_rental_counts AS t1
INNER JOIN customer_total_rentals AS t2
  ON t1.customer_id = t2.customer_id
INNER JOIN avg_category_rental_counts AS t3
  ON t1.category_name = t3.category_name
INNER JOIN customer_category_percentiles AS t4
  ON t1.customer_id = t4.customer_id
  AND t1.category_name = t4.category_name;

-- inspect customer_id = 1 rows sorted by percentile
SELECT *
FROM customer_category_joint_table
WHERE customer_id = 1
ORDER BY percentile;
```
Output:

|customer_id   |category_name |	rental_count  |	total_rental_count |avg_rental_count |	percentile  |
|--------------|--------------|---------------|--------------------|-----------------|--------------|
|       1	   | Classics	  |     6	      |       32	       |    2.006410256	 |      1       |
|       1	   | Comedy	      |     5	      |       32	       |    1.901010101	 |      1       |
|       1	   | Drama	      |     4	      |       32	       |    2.115768463	 |      3       |
|       1	   | Music	      |     2	      |       32	       |    1.856823266	 |      21      |
|       1	   | New	      |     2	      |       32	       |    2.008547009	 |      27      |
|       1	   | Sci-Fi	      |     2	      |       32	       |    2.171597633	 |      31      |
|       1	   | Action	      |     2	      |       32	       |    2.180392157	 |      34      |
|       1	   | Sports	      |     2	      |       32	       |    2.271676301	 |      35      |
|       1	   | Animation	  |     2	      |       32	       |    2.332	     |      39      |
|       1	   | Travel	      |     1	      |       32	       |    1.893665158	 |      58      |
|       1	   | Games	      |     1	      |       32	       |    2.044303797	 |      61      |
|       1	   | Foreign	  |     1	      |       32	       |    2.095334686	 |      62      |
|       1	   | Documentary  |     1	      |       32	       |    2.173913043	 |      65      |
|       1	   | Family	      |     1	      |       32	       |    2.18762475	 |      66      |

Now fields like avg_comparison and category_percentage need to be added. Lets incorporate those fields as well


###   4.3.6 Adding Calculated fields
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

The field avg_comparison calcualted the difference between the total number of films watched by the customer minus the average viewrship in that category. The category percentage field describes the proportion of films watched by the customer(in each category) with respect to the total films he/she as viewed.

``` SQL
DROP TABLE IF EXISTS customer_category_joint_table;
CREATE TEMP TABLE customer_category_joint_table AS
SELECT
  t1.customer_id,
  t1.category_name,
  t1.rental_count,
  t1.latest_rental_date,
  t2.total_rental_count,
  t3.avg_rental_count,
  t4.percentile,
  t1.rental_count - t3.avg_rental_count AS average_comparison,
  -- round to nearest integer for percentage after multiplying by 100
  ROUND(100 * t1.rental_count / t2.total_rental_count) AS category_percentage
FROM category_rental_counts AS t1
INNER JOIN customer_total_rentals AS t2
  ON t1.customer_id = t2.customer_id
INNER JOIN average_category_rental_counts AS t3
  ON t1.category_name = t3.category_name
INNER JOIN customer_category_percentiles AS t4
  ON t1.customer_id = t4.customer_id
  AND t1.category_name = t4.category_name;


-- See the records for customer_id = 1
SELECT *
FROM customer_category_join_table
WHERE customer_id = 1
ORDER BY percentile
LIMIT 5;

```

Output:

|customer_id |	category_name |	rental_count| 	latest_rental_date	    | total_rental_count |	avg_rental_count |	percentile | average_comparison    |category_percentage |
|------------|----------------|------------ |-------------------------- |--------------------|-------------------|-------------|-----------------------|--------------------|
|       1	 |    Comedy	  |   5	        |  2005-08-22T19:41:37.000Z	|  32	             |       1	         |   1	       |     4	               |        16          |
|       1	 |    Classics	  |   6	        |  2005-08-19T09:55:16.000Z	|  32	             |       2	         |   1	       |     4	               |        19          |
|       1	 |    Drama	      |   4	        |  2005-08-18T03:57:29.000Z	|  32	             |       2	         |   3	       |     2	               |        13          |
|       1	 |    Music	      |   2	        |  2005-07-09T16:38:01.000Z	|  32	             |       1	         |   21	       |     1	               |        6           |
|       1	 |    New	      |   2	        |  2005-08-19T13:56:54.000Z	|  32	             |       2	         |   27	       |     0	               |        6           |


###   Ordering and Filtering Rows with ROW_NUMBER
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Since we need only the top 2 category counts for each customer, we shall apply the window function ROW_NUMBER() along with OVER() clause where we would partiton by customer id and order by rental_count along with latest_rental_date. We would achieve all this by using the data created from the previous table.

``` sql
DROP TABLE IF EXISTS top_categories_information;

-- Note that you need an extra pair of (brackets) when you create tables
-- with CTEs inside the SQL statement!
CREATE TEMP TABLE top_categories_information AS (
-- use a CTE with the ROW_NUMBER() window function implemented
WITH ordered_customer_category_joint_table AS (
  SELECT
    customer_id,
    ROW_NUMBER() OVER (
      PARTITION BY customer_id
      ORDER BY rental_count DESC, latest_rental_date DESC
    ) AS category_ranking,
    category_name,
    rental_count,
    average_comparison,
    percentile,
    category_percentage
  FROM customer_category_joint_table
)
-- filter out top 2 rows from the CTE for final output
SELECT *
FROM ordered_customer_category_joint_table
WHERE category_ranking <= 2
);
```

Output:

|customer_id |	category_ranking |	category_name |	rental_count|	average_comparison|	percentile |category_percentage|
|------------|-------------------|----------------|-------------|---------------------|------------|-------------------|
|1	         |      1	         | Classics	      |   6	        |        4	          |  1	       |    19             |
|1	         |      2	         | Comedy	      |   5	        |        4	          |  1	       |    16             |
|2	         |      1	         | Sports	      |   5	        |        3	          |  3	       |    19             |
|2	         |      2	         | Classics	      |   4	        |        2	          |  2	       |    15             |
|3	         |      1	         | Action	      |   4	        |        2	          |  5	       |    15             |
|3	         |      2	         | Sci-Fi	      |   3	        |        1	          |  15	       |    12             |
								 
Now at least we have ticked off a few checkboxes for the requirements with respect to the email template.


 





								   





