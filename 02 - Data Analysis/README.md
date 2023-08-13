# 2. Data Analysis

## 2.1 Final State Layout

     The main columns which we would be working with in order to generate valuable insights for the email template given to us are shown below:

   - category_name: The name of the top 2 ranking categories
   - rental_count: How many total films have they watched in this category
   - average_comparison: How many more films has the customer watched compared to the average DVD Rental Co customer
   - percentile: How does the customer rank in terms of the top X% compared to all other customers in this film category?
   - category_percentage: What proportion of total films watched does this category make up?
   
 
 And the final output should look like this:
 
 Output:

|customer_id  |category_ranking |	category_name |	rental_count |	average_comparison | percentile  | category_percentage|
|-------------|-----------------|-----------------|--------------|---------------------|-------------|--------------------|
|1 			  |			1 		|	Classics 	  |    6         |      4              |  1          |    19              |
|1 			  |			2 		|	Comedy        |    2         |      3              |  2          |    16              |
|2 			  |			1 		|	Sports        |    6         |      4              |  11         |     17             |
|2 			  |			2 		|	Classics 	  |    5         |      9              |  20         |     16             |
|3 			  |			1 		|	Action        |    1         |      4              |  14         |     18             |


And so on ........

## 2.2 Reverse Engineering

As we can see from the above output table the ``` rental_count ```  is an important column and with that it would lead us to calculating columns like ``` average_comparison ``` and ``` percentile ``` for each customer.

Also we need to find the top two categories for each customer along with the category  name, Something like this:

|customer_id |	category_name  |rental_count|
|------------|-----------------|------------|
|1 			|		Classics   |     6      |
|1 			|		Comedy 	   |     5      |
|2 			|		Sports 	   |     5      |
|2 			|		Classics   |     4      |


## 2.3 Mapping the join journey

Based on our initial data exploration and view of the ERD diagram the layout of the  Join Journey is shown as follows:


|Join Journey Part   |	Start 	      |    End 	      | Foreign Key |
|--------------------|----------------|---------------|------------ |
|Part 1              |	rental 	      |  inventory 	  |inventory_id |
|Part 2              |	inventory 	  |  film 	      |  film_id    |
|Part 3              |	film 	      |  film_category|	 film_id    |
|Part 4              |	film_category |	category 	  | category_id |

In order to find the ```rental_value``` at a customer_id level we need 2 columns namely: 

    1. ```customer_id``` 
    2. ```category_name```

The rental table is the main table that we will start from as it is the only table that contains customer_id. We would need to follow the journey path to reach the category table and include the category name. The last two tables containing actor details will be skipped for now.


## 2.4 Deciding which type of joins to use

We can define our purpose by first answering this basic question: 

> How can we  generate the rental_count calculation - the number of films that a customer has watched in a specific category.


When we look through the other tables in the ERD - we can notice the inventory table which can be used to help us get the film_id column for each rental.

We can think of the deeper purpose of this join between the two tables in the following way:

> We need to keep all of the customer rental records from dvd_rentals.rental and match up each record with its equivalent film_id value from the dvd_rentals.inventory table.

There are two type of joins we can think of viz, Left Join and Inner Join. Let's dig up a bit further to decide which join suits for our problem solving more.

### 2.4.1 Key Analytical questions

The two questions that we need to lookup before deciding which join to use are:

     1. How many records exist per ```inventory_id``` value in ```rental``` or ```inventory tables```?
     2. How many overlapping and missing unique ```foreign key``` values are there between the two tables?
	 
Now applying the two-phase approach that we shall use to answer the above queestions. First, generating some hypotheses about the data and then try to validate it to see if we are correct.

### 2.4.2 Generating hypothesis while looking at the data

Since we know that the rental table contains every single rental for each customer - it makes sense logically that each valid rental record in the rental table should have a relevant inventory_id record as people need to physically hire some item in the store.

Additionally - it also makes sense that a specific item might be rented out by multiple customers at different times as customers return the DVD as shown by the return_date column in the dataset.

Now when we think about the inventory table - every item should have a unique inventory_id but there may also be multiple copies of a specific film.

From these 2 key pieces of real life insight - we can generate some hypotheses about our 2 datasets.

     1. The number of unique ```inventory_id``` records will be equal in both ```dvd_rentals.rental``` and ```dvd_rentals.inventory``` tables.
     2. There will be a multiple records per unique ```inventory_id``` in the dvd_rentals.rental table.
     3. There will be multiple ```inventory_id``` records per unique ```film_id``` value in the ```dvd_rentals.inventory``` table.
	 
### 2.4.3 Validating the hypothesis using the given data

We can use SQL to solve for these hypothesis.

#### 2.4.3.1 Hypothesis 1
    
The number of unique ```inventory_id``` records will be equal in both ```dvd_rentals.rental``` and ```dvd_rentals.inventory``` tables
	
First we shall check  the unique number of inventory_id present in the retal table.
	
```sql 
select count(distinct inventory_id)
   from dvd_rentals.rental
```
Output:

|count|
|-----|
|4580 |

Now lets take a look at the inventory table for the same. 
	
	
 ```sql 
select count(distinct inventory_id)
   from dvd_rentals.rental
```
Output:

|count|
|-----|
|4581 |
	
We can observe that there is one count extra compared to the total count returned from the rental table. This result **invalidates** our hypothesis. Looks like there was an inventrory_id which was never used for lending to the customer.
	
#### 2.4.3.2 Hypothesis 2

    There will be a multiple records per unique ```inventory_id``` in the dvd_rentals.rental table.
	
	```SQL
	-- first generate group by counts on the target_column_values column
	WITH counts_base AS (
                          SELECT
                            inventory_id AS target_column_values,
                            COUNT(*) AS row_counts
                          FROM dvd_rentals.rental
                          GROUP BY target_column_values
                          )
    -- summarize the group by counts above by grouping again on the row_counts from counts_base CTE part
        SELECT
          row_counts,
          COUNT(target_column_values) as count_of_target_values
        FROM counts_base
        GROUP BY row_counts
        ORDER BY row_counts;
	```
	Output:
	
    |row_counts| count_of_target_values |
	|----------|------------------------|
    |   1	   |       4                |
    |   2	   |       1126             |
    |   3	   |       1151             |
    |   4	   |       1160             |
    |   5	   |       1139             |

	The above table confirms our 2nd hypothesis.
	
#### 2.4.3.3 Hypothesis 3
	
	 There will be multiple ```inventory_id``` records per unique ```film_id``` value in the ```dvd_rentals.inventory``` table.
	
	```SQL
	with counts_base as (
                          select film_id as target_column_values,
                                 COUNT(DISTINCT inventory_id) AS unique_record_counts
                                 from dvd_rentals.inventory
                                 GROUP BY target_column_values
                          )
    SELECT unique_record_counts,
    COUNT(target_column_values) as count_of_target_values
    from counts_base
    group by unique_record_counts
	```
	Output:
	
	|target_column_values| unique_record_counts|
	|-------------------|--------------------|
    |             1	    |       8            |
    |             2	    |       3            |
    |             3	    |       4            |
    |             4	    |       7            |
    |             5	    |       3            |
    |             6	    |       6            |
    |             7	    |       5            |
    |             8	    |       4            |
    |             9	    |       5            |
    |             10	|       7            |

	We can confirm that there are indeed multiple unique inventory_id per film_id value in the dvd_rentals.inventory table.
	
#### 2.3.5 Returning to the 2 Key Questions

     1. How many records exist per inventory_id value in rental or inventory tables?
	 2. How many overlapping and missing unique foreign key values are there between the two tables?
	 
	 
	 *** rental distribution analysis on inventory_id foreign key ***
	 
	 ``` SQL
	 -- first generate group by counts on the foreign_key_values column
        WITH counts_base AS (
        SELECT
          inventory_id AS foreign_key_values,
          COUNT(*) AS row_counts
        FROM dvd_rentals.rental
        GROUP BY foreign_key_values
        )
        -- summarize the group by counts above by grouping again on the row_counts from counts_base CTE part
        SELECT
          row_counts,
          COUNT(foreign_key_values) as count_of_foreign_keys
        FROM counts_base
        GROUP BY row_counts
        ORDER BY row_counts;
	```
	
	Output:
	
	|row_counts| count_of_foreign_keys  |
	|----------|------------------------|
    |   1	   |       4                |
    |   2	   |       1126             |
    |   3	   |       1151             |
    |   4	   |       1160             |
    |   5	   |       1139             |
	
	From the above output it can be observed that there is a 1 to many mapping between inventory_id and records in the rental table. Also it can be noted that 4 inventory_id's have only a single count here. 1126 inventory_ids exist in 2 different rows, 1151 inventory_ids exixst in 3 different rows and so on
	
	*** inventory distribution analysis on inventory_id foreign key***
	
	```SQL
	WITH counts_base AS (
                          SELECT
                            inventory_id AS foreign_key_values,
                            COUNT(*) AS row_counts
                          FROM dvd_rentals.inventory
                          GROUP BY foreign_key_values
                          )
    SELECT
      row_counts,
      COUNT(foreign_key_values) as count_of_foreign_keys
    FROM counts_base
    GROUP BY row_counts
    ORDER BY row_counts;
	```
	
	
	Output:
	
	|row_counts|count_of_foreign_keys|
	|----------|---------------------|
	|	1	   |      4581           |
	
	From the above output we can confirm that there is a 1:1 mapping betwen the inventory_id and each table row record in the inventory table
	
	
	Proceeding to the second question
	
	**How many overlapping and missing unique foreign key values are there between the two tables?**
	
	
	Proceeding to check how many foreign keys only exist in the rental table and not in the inventory table:
	
	``` sql 
			SELECT
			COUNT(DISTINCT rental.inventory_id)
			FROM dvd_rentals.rental
			WHERE NOT EXISTS (
					SELECT inventory_id
					FROM dvd_rentals.inventory
					WHERE rental.inventory_id = inventory.inventory_id);
	```
	
	Output:
	
	|count|
    |-----|
    | 0   |
	
	Great we can confirm that there are no inventory_id records which appear in the dvd_rentals.rental table which does not appear in the dvd_rentals.inventory table.
	
	Proceeding to check how many foreign keys only exist in the rental table and not in the inventory table:
	
	``` SQL
			-- how many foreign keys only exist in the right table and not in the left?
			-- note the table reference changes
			SELECT
			COUNT(DISTINCT inventory.inventory_id)
			FROM dvd_rentals.inventory
			WHERE NOT EXISTS (
			SELECT inventory_id
			FROM dvd_rentals.rental
			WHERE rental.inventory_id = inventory.inventory_id
			);
	```
	
	Output:
	
	|count|
    |-----|
    | 1   |
	
	Ok - we’ve spotted a single inventory_id record. Let’s inspect further:
	
	``` SQL
	SELECT *
    FROM dvd_rentals.inventory
    WHERE NOT EXISTS (
      SELECT inventory_id
      FROM dvd_rentals.rental
      WHERE rental.inventory_id = inventory.inventory_id
    );
	```
	
	Output:
	
	|inventory_id |	film_id |	store_id  | last_update             |
	|-------------|---------|-------------|-------------------------|                         
    |   5	      |     1	|  2	      | 2006-02-15T05:09:17.000Z|

	This single record might seem off at first - but let’s revisit what the inventory data actually represents.

     It is linked to a specific film record which could be rented out by a customer. One such reason for this odd record could be that this specific rental inventory unit was never rented out by a customer.
	 
    Checking the intersection of foreign key(inventory_id) between the two tables.
	
	``` SQL
         SELECT
               COUNT(DISTINCT inventory_id) 
        FROM dvd_rentals.rental
        WHERE EXISTS (
            SELECT inventory_id
            FROM dvd_rentals.inventory
            WHERE rental.inventory_id = inventory.inventory_id
                       );
    ```
	Output:
	
	|count|
    |-----|
    |4580 |
	

	 
Now, that we have analyzed and checked, let's proceed on to the Table Joining part.









