<p align="center">
    <img src="Images\Marketing_Analytics_Case_Study_Banner.png" alt="marketing-analytics">
</p>

# Marketing Analytics Case Study -  Serious SQL


This is a marketing analytics case study taken from the Serious SQL course by Danny Ma. The marketing team have shared with us a draft of the email they wish to send to their customers.

## 📃 Table of Contents


- [🤔 About](#about)

- [🔭 Requirements Overview](#requirements_overview)
  +  [Requirement #1](#requirement_1)
  +  [Requirement #2](#requirement_2)
  +  [Requirement #3 & #4](#requirement_3_4)
  +  [Requirement #5](#requirement_5)

- [🔍 Data Exploration](#data_exploration)
- [🔬 Data Analysis](#data_analysis)
- [🧲 Join Implementation](#join_implementation)
- [🏗️ Problem Solving](#problem_solving)
- [🏁 Final Solution](#final_solution)
- [🪙 Business Questions](#business_questions)
- [Support]

## 🤔 About  <a name = "about"></a>

Personalized customer emails based off marketing analytics is a winning formula for many digital companies, and this is exactly the initiative that the leadership team at DVD Rental Co has decided to tackle!

We have been asked to support the customer analytics team at DVD Rental Co who have been tasked with generating the necessary data points required to populate specific parts of this first-ever customer email campaign.

We will be using SQL to solve the various business problems that the customer analytics team needs to answer.

## 🔭 Requirements Overview <a name = "requirements_overview"></a>

The marketing team at  DvD Rental Co has shared with a draft of the email template. The various business requirements can be found in this template and data points need to be gathered for every customer

<p align="center">
    <img src="Images\Requirements_Overview.png" alt="email-template" width="500px">
</p>

<p align="center"> <u>Source: <a href="https://www.datawithdanny.com/">Serious SQL</a></u>
    <br> 
</p>
															


## Requirement 1️⃣    <a name = "requirement_1"></a>

For each customer, we need to identify the top 2 categories for each customer based off their past rental history. 


<br>

<p align="center">
    <img src="Images\Requirement_1.png" alt="email-template" width="500px">
</p>

<p align="center"> <u>Source: <a href="https://www.datawithdanny.com/">Serious SQL</a></u>
    <br> 
</p>
															 
															 
## Requirement 2️⃣     <a name = "requirement_2"></a>

The marketing team has requested that we recommend 3 popular films  from each customers top 2 categories keeping in mind that the films that are recomended are films which the customer has not watched before


<p align="center">
    <img src="Images\Requirement_2.png" alt="email-template" width="500px">
</p>

<p align="center"> <u>Source: <a href="https://www.datawithdanny.com/">Serious SQL</a></u>
    <br> 
</p>
															
															
## Requirement 3️⃣ & 4️⃣     <a name = "requirement_3_4"></a>

Individual Customer Insights

The number of films watched by each customer in their top 2 categories is required as well as some specific insights.

> For the 1st category, the marketing requires the following insights (requirement 3):

    1. How many total films have they watched in their top category?
    2. How many more films has the customer watched compared to the average DVD Rental Co customer?
    3. How does the customer rank in terms of the top X% compared to all other customers in this film category?

> For the second ranking category (requirement 4):

    1. How many total films has the customer watched in this category?
    2. What proportion of each customer’s total films watched does this count make?
	
<br>
	   
<p align="center">
    <img src="Images\Requirement_3_4.png" alt="email-template" width="500px">
</p>

<p align="center"> <u>Source: <a href="https://www.datawithdanny.com/">Serious SQL</a></u>
    <br> 
</p>
	
	
## Requirement 5️⃣    <a name = "requirement_5"></a>

Favorite Actor Recommendations

Along with the top 2 categories, marketing has also requested top actor film recommendations where up to 3 more films are included in the recommendations list as well as the count of films by the top actor.

<br>
	   
<p align="center">
    <img src="Images\Requirement_5.png" alt="email-template" width="500px">
</p>

<p align="center"> <u>Source: <a href="https://www.datawithdanny.com/">Serious SQL</a></u>
    <br> 
</p>
															
															
## 🔍 Data Exploration <a name = "data_exploration"></a>

There are 7 tables in this case study and this can be seen in the ERD diagram below. The seven tables are rental, inventory, film, film_category, category, film_actor, actor.

<br>
	   
<p align="center">
    <img src="Images\ERD_diagram.png" alt="email-template" width="500px">
</p>

<p align="center"> <u>Source: <a href="https://www.datawithdanny.com/">Serious SQL</a></u> 
    <br> 
</p>
															
### Click here to view ⬇️:
[![forthebadge](/Images/badges/solution-data-exploration.svg)](https://github.com/BalakumaranRKB/Marketing_Analytics_Case_Study/blob/main/01-%20Data%20Exploration)



## 🔬 Data Analysis <a name = "data_analysis"></a>

After checking out the dataset, we start looking at the important columns and come up with a few ideas to really understand the data better. When we analyze everything, we figure out that for our example, it doesn't matter whether we use an INNER JOIN or LEFT JOIN because all the values in our left table are already in the target table.
<br>
	   
															
### Click here to view ⬇️:
[![forthebadge](/Images/badges/solution-data-analysis.svg)](https://github.com/BalakumaranRKB/Marketing_Analytics_Case_Study/tree/main/02%20-%20Data%20Analysis)


## 🧲 Join Implementation <a name = "join_implementation"></a>

After that, we begin applying the table joins, which will assist us in tackling the problem. Based on our analysis, we've decided on the following sequence for joining the tables.

<br>


Taking a look at our table joining journey:

|Join Journey Part   |	Start 	      |    End 	      | Foreign Key |
|--------------------|----------------|---------------|------------ |
|Part 1              |	rental 	      |  inventory 	  |inventory_id |
|Part 2              |	inventory 	  |  film 	      |  film_id    |
|Part 3              |	film 	      |  film_category|	 film_id    |
|Part 4              |	film_category |	category 	  | category_id |
	   
															
### Click here to view ⬇️:
[![forthebadge](/Images/badges/solution-join-implementation.svg)](https://github.com/BalakumaranRKB/Marketing_Analytics_Case_Study/tree/main/03-%20Join%20Implementation)



## 🏗️ Problem Solving <a name = "problem_solving"></a>

Having completed the process of joining all tables to create our base table, we can now shift our focus towards addressing business requirements 1 - 4. The resulting output at the conclusion of this phase is presented below. As demonstrated, this outcome effectively fulfills several of the business requirements outlined in the email template.

<br>
Output:

|customer_id |	category_ranking |	category_name |	rental_count|	average_comparison|	percentile |category_percentage|
|------------|-------------------|----------------|-------------|---------------------|------------|-------------------|
|1	         |      1	         | Classics	      |   6	        |        4	          |  1	       |    19             |
|1	         |      2	         | Comedy	      |   5	        |        4	          |  1	       |    16             |
|2	         |      1	         | Sports	      |   5	        |        3	          |  3	       |    19             |
|2	         |      2	         | Classics	      |   4	        |        2	          |  2	       |    15             |
|3	         |      1	         | Action	      |   4	        |        2	          |  5	       |    15             |
|3	         |      2	         | Sci-Fi	      |   3	        |        1	          |  15	       |    12             |
															
### Click here to view ⬇️:
[![forthebadge](/Images/badges/solution-join-implementation.svg)](https://github.com/BalakumaranRKB/Marketing_Analytics_Case_Study/tree/main/04-Problem%20Solving)


## 🏁 Final Solution <a name = "Final_Solution"></a>


Lastly, we began putting the final solutions in place for each of the requirements. Now, let's take a brief look at what our tables are showing us.

<br>

## 👉🏾 Requirement #1
Below, you can observe the top two categories attributed to each customer: (For now, we'll focus on the output for customer_id = 1, 2)

 Output:
 
 |customer_id |category_name |   rental_count|   category_rank   |
 |------------|--------------|---------------|-------------------|
 |     1	  |     Classics |      6	     |       1           |
 |     1	  |     Comedy	 |      5	     |       2           |
 |     2	  |     Sports	 |      5	     |       1           |
 |     2	  |     Classics |      4	     |       2           |
 |     3	  |     Action	 |      4	     |       1           |

## 👉🏾 Requirement #2

The film recommendations for customer_id = 1 are as follows:

Output:

|customer_id |  category_name | category_rank |	film_id	|        title	         |rental_count	| reco_rank |
|------------|--------------- |---------------|---------|----------------------- |--------------|-----------|
|      1	 |    Classics	  |       1	      | 891	    |   TIMBERLAND SKY	     |    31	    |    1      |
|      1	 |    Classics	  |       1	      | 358	    |   GILMORE BOILED	     |    28	    |    2      |
|      1	 |    Classics	  |       1	      | 951	    |   VOYAGE LEGALLY	     |    28	    |    3      |
|      1	 |    Comedy	  |       2	      | 1000	|   ZORRO ARK	         |    31	    |    1      |
|      1	 |    Comedy	  |       2	      | 127	    |   CAT CONEHEADS	     |    30	    |    2      |
|      1	 |    Comedy	  |       2	      | 638	    |   OPERATION OPERATION	 |    27	    |    3      |




## 👉🏾 Requirement #3& #4:

For the top first category, here is how the insights look like:

|customer_id | category_name  |	rental_count| average_comparsion |  percentile |
|----------- |----------------|-------------|--------------------|-------------|
|     323	 |    Action	  |        7	|         5	         |      1      |
|     506	 |    Action	  |        7	|         5	         |      1      |
|     151	 |    Action	  |        6	|         4	         |      1      |
|     410	 |    Action	  |        6	|         4	         |      1      |
|     126	 |    Action	  |        6	|         4	         |      1      |

And, for the second top category, the output table is:

Output:		
|customer_id |	category_name |	rental_count |	total_percentage |
|------------|----------------|--------------|-------------------|
|  184	     |  Drama	      |   3	         |   13              |
|  87	     |  Sci-Fi	      |   3	         |   10              |
|  477	     |  Travel	      |   3	         |   14              |
|  273	     |  New	          |   4	         |   11              |
|  550	     |  Drama	      |   4	         |   13              |



## 👉🏾 Requirement #5

The favorite actor for each customer can be found in the temp table top_actor_counts:

Output:

|customer_id |	actor_id |	first_name |  last_name   |	rental_count |
|------------|-----------|-------------|--------------|------------- |
|      1	 |    37	 |  VAL	       |  BOLGER	  |     6	     | 
|      2	 |    107	 |  GINA	   |  DEGENERES	  |     5	     | 
|      3	 |    150	 |  JAYNE	   |  NOLTE	      |     4	     | 
|      4	 |    102	 |  WALTER	   |  TORN	      |     4	     | 
|      5	 |    12	 |  KARL	   |  BERRY	      |     4	     | 
|      6	 |    191	 |  GREGORY	   |  GOODING	  |     4	     | 
|      7	 |    65	 |  ANGELA	   |  HUDSON	  |     5	     | 
|      8	 |    167	 |  LAURENCE   |  BULLOCK	  |     5	     | 
|      9	 |    23	 |  SANDRA	   |  KILMER	  |     3	     | 
|      10	 |    12	 |  KARL	   |  BERRY	      |     4	     |
			  

### Click here to view ⬇️:
[![forthebadge](/Images/badges/solution-join-implementation.svg)](https://github.com/BalakumaranRKB/Marketing_Analytics_Case_Study/tree/main/05-Final%20Solution)



## 🪙 Business Questions <a name = "business_questions"></a>

The following questions are part of the final case study quiz - these are example questions the Marketing team might be interested in!

1. Which film title was the most recommended for all customers?
2. How many customers were included in the email campaign?
3. Out of all the possible films - what percentage coverage do we have in our recommendations?
4. What is the most popular top category?
5. What is the 4th most popular top category?
6. What is the average percentile ranking for each customer in their top category rounded to the nearest 2 decimal places?
7. What is the median of the second category percentage of entire viewing history?
8. What is the 80th percentile of films watched featuring each customer’s favourite actor?
9. What was the average number of films watched by each customer?
10. What is the top combination of top 2 categories and how many customers if the order is relevant (e.g. Horror and Drama is a different combination to Drama and Horror)
11. Which actor was the most popular for all customers?
12. How many films on average had customers already seen that feature their favourite actor rounded to closest integer?
13. What is the most common top categories combination if order was irrelevant and how many customers have this combination? (e.g. Horror and Drama is a the same as Drama and       Horror)

### Click here to view ⬇️:
[![forthebadge](/Images/badges/solution-join-implementation.svg)](https://github.com/BalakumaranRKB/Marketing_Analytics_Case_Study/tree/main/06-Solving%20Business%20Questions)




