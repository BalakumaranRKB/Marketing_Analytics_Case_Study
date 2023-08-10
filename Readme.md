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

- [Data Exploration]
- [Data Analysis]
- [Join Implementation]
- [Problem Solving]
- [Final Solution]
- [Business Questions]
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
															
															
## 🔍 Data Exploration <a name = "data exploration"></a>

There are seven tables in this case study and this can be seen in the ERD diagram below. The seven tables are rental, inventory, film, film_category, category, film_actor, actor.

<br>
	   
<p align="center">
    <img src="Images\ERD_diagram.png" alt="email-template" width="500px">
</p>

<p align="center"> <u>Source: <a href="https://www.datawithdanny.com/">Serious SQL</a></u>
    <br> 
</p>
															
Click here to view :



