1. Data Exploration
=========================================================================

1.1 Rental Table
-------------------------------------------------------------------------
This is the starting table in our database. Showing the first 10 rows  for this table:

```sql
select * from dvd_rentals.rental
limit 10
```



Output:

|rental_id  | rental_date  |	inventory_id |	customer_id |	return_date          | 	staff_id | last_update|
| --------- | ------------ | ----------------|--------------|------------------------|------|--------------|
|1 |2005-05-24T22:53:30.000Z	|  367  	 | 130			|2005-05-26T22:04:30.000Z|	1	|2006-02-15T21:30:53.000Z|
|2 |2005-05-24T22:54:33.000Z	| 1525		 | 459			|2005-05-28T19:40:33.000Z|	1	|2006-02-15T21:30:53.000Z|
|3 |2005-05-24T23:03:39.000Z	| 1711		 | 408			|2005-06-01T22:12:39.000Z|	1	|2006-02-15T21:30:53.000Z|
|4 |2005-05-24T23:04:41.000Z	| 2452		 | 333			|2005-06-03T01:43:41.000Z|	2	|2006-02-15T21:30:53.000Z|
|5 |2005-05-24T23:05:21.000Z	| 2079		 | 222			|2005-06-02T04:33:21.000Z|	1	|2006-02-15T21:30:53.000Z|
|6 |2005-05-24T23:08:07.000Z	| 2792		 | 549			|2005-05-27T01:32:07.000Z|	1	|2006-02-15T21:30:53.000Z|
|7 |2005-05-24T23:11:53.000Z	| 3995		 | 269			|2005-05-29T20:34:53.000Z|	2	|2006-02-15T21:30:53.000Z|
|8 |2005-05-24T23:31:46.000Z	| 2346		 | 239			|2005-05-27T23:33:46.000Z|	2	|2006-02-15T21:30:53.000Z|
|9 |2005-05-25T00:00:40.000Z	| 2580		 | 126			|2005-05-28T00:22:40.000Z|	1	|2006-02-15T21:30:53.000Z|
|10|2005-05-25T00:02:21.000Z	| 1824		 | 399			|2005-05-31T22:44:21.000Z|	2	|2006-02-15T21:30:53.000Z|

The rental_id column is a primary column. Let's see how many rows are present for this table:

```sql
select count(*) from dvd_rentals.rental
```

Output:

|count|
|------|
| 16044 |

From the above output it can be observed that there are around 16000 records.
From the ERD diagram we can observe that there is a linkage between rental and inventory table through the inventory_id column.

1.2 Inventory Table
------------------------------------------------------------------------------------------------------------------------
The inventory table consists of inventory_id,film_id, store_id and last_update columns. Showing the first 10 rows  for this table:

```sql
select * from dvd_rentals.inventory
limit 10
```

Output:

|inventory_id 		|	film_id	|store_id |	last_update|
|------------------	|---------|---------|------------|
|1			  		| 1	    | 1	      | 2006-02-15T05:09:17.000Z       |
|2			  		| 1	    | 1	      | 2006-02-15T05:09:17.000Z       |
|3			  		| 1	    | 1	      | 2006-02-15T05:09:17.000Z       |
|4			  		| 1	    | 1	      | 2006-02-15T05:09:17.000Z       |
|5			  		| 1	    | 2	      | 2006-02-15T05:09:17.000Z       |
|6			  		| 1	    | 2	      | 2006-02-15T05:09:17.000Z       |
|7			  		| 1	    | 2	      | 2006-02-15T05:09:17.000Z       |
|8			  		| 1	    | 2	      | 2006-02-15T05:09:17.000Z       |
|9			  		| 2	    | 2	      | 2006-02-15T05:09:17.000Z       |
|10			  		| 2	    | 2	      | 2006-02-15T05:09:17.000Z       |


From the above output we notice that for film_id = 1 there are multiple inventory_id's linked to it. A single film can have multiple copies spread out across different stores
Here the inventory_id is the primary column for this table. Let's see how many rows are present for this table:

```sql
select count(*) from dvd_rentals.inventory
limit 10
```

Output:

|count|
|-----|
| 4581 |

From the above output it can be observed that there are around  4000 records.
From the ERD diagram we can observe that there is a linkage between the inventory  and film table through the film_id column.


1.3 Film Table
------------------------------------------------------------------------------------------------------------------------
The Film table consists of 14 columns with *film_id* being the primary column. Showing the first 10 rows  for this table:

```sql
select * from dvd_rentals.film
limit 10
```

Output:

film_id	 | title |	description	| release_year	| language_id	| original_language_id |	rental_duration	| rental_rate	| length |	replacement_cost |	rating	|last_update |	special_features |	fulltext |
|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|
| 1|	ACADEMY DINOSAUR|	A Epic Drama of a Feminist And a Mad Scientist who must Battle a Teacher in The Canadian Rockies|	2006|	1|		6|	0.99	|86	|20.99|	PG	|2006-02-15T05:03:42.000Z	|Deleted Scenes,Behind the Scenes|	'academi':1 'battl':15 'canadian':20 'dinosaur':2 'drama':5 'epic':4 'feminist':8 'mad':11 'must':14 'rocki':21 'scientist':12 'teacher':17|
| 2|	ACE GOLDFINGER	|A Astounding Epistle of a Database Administrator And a Explorer who must Find a Car in Ancient China	2006 |	1	|	3 |	4.99 |	48 |	12.99 |	G |	2006-02-15T05:03:42.000Z |	Trailers,Deleted Scenes|	'ace':1 'administr':9 'ancient':19 'astound':4 'car':17 'china':20 'databas':8 'epistl':5 'explor':12 'find':15 'goldfing':2 'must':14|
| 3|	ADAPTATION HOLES|	A Astounding Reflection of a Lumberjack And a Car who must Sink a Lumberjack in A Baloon Factory	2006 |	1	|	7 |	2.99 |	50 |	18.99 |	NC-17|	2006-02-15T05:03:42.000Z|	Trailers,Deleted Scenes|	'adapt':1 'astound':4 'baloon':19 'car':11 'factori':20 'hole':2 'lumberjack':8,16 'must':13 'reflect':5 'sink':14|
| 4|	AFFAIR PREJUDICE|	A Fanciful Documentary of a Frisbee And a Lumberjack who must Chase a Monkey in A Shark Tank|	2006 |	1	|	5 |	2.99 |	117 |	26.99|	G|	2006-02-15T05:03:42.000Z |	Commentaries,Behind the Scenes|	'affair':1 'chase':14 'documentari':5 'fanci':4 'frisbe':8 'lumberjack':11 'monkey':16 'must':13 'prejudic':2 'shark':19 'tank':20|
| 5|	AFRICAN EGG|	A Fast-Paced Documentary of a Pastry Chef And a Dentist who must Pursue a Forensic Psychologist in The Gulf of Mexico |	2006 |	1	 |	6 |	2.99 |	130|	22.99|	G |	2006-02-15T05:03:42.000Z	|Deleted Scenes|	'african':1 'chef':11 'dentist':14 'documentari':7 'egg':2 'fast':5 'fast-pac':4 'forens':19 'gulf':23 'mexico':25 'must':16 'pace':6 'pastri':10 'psychologist':20 'pursu':17|
| 6|	AGENT TRUMAN	|A Intrepid Panorama of a Robot And a Boy who must Escape a Sumo Wrestler in Ancient China	2006 |	1	|	3 |	2.99 |	169	 | 17.99	|PG	|2006-02-15T05:03:42.000Z |	Deleted Scenes|	'agent':1 'ancient':19 'boy':11 'china':20 'escap':14 'intrepid':4 'must':13 'panorama':5 'robot':8 'sumo':16 'truman':2 'wrestler':17
| 7|	AIRPLANE SIERRA	|A Touching Saga of a Hunter And a Butler who must Discover a Butler in A Jet Boat|	2006 |	1 |		6 |	4.99|	62 |	28.99 |	PG-13 |	2006-02-15T05:03:42.000Z	|Trailers,Deleted Scenes|	'airplan':1 'boat':20 'butler':11,16 'discov':14 'hunter':8 'jet':19 'must':13 'saga':5 'sierra':2 'touch':4
| 8|	AIRPORT POLLOCK	|A Epic Tale of a Moose And a Girl who must Confront a Monkey in Ancient India |	2006|	1	|	6|	4.99 |	54	|15.99	|R |	2006-02-15T05:03:42.000Z	|Trailers |	'airport':1 'ancient':18 'confront':14 'epic':4 'girl':11 'india':19 'monkey':16 'moos':8 'must':13 'pollock':2 'tale':5|
| 9|	ALABAMA DEVIL	|A Thoughtful Panorama of a Database Administrator And a Mad Scientist who must Outgun a Mad Scientist in A Jet Boat |	2006 |	1	|	3 |	2.99 |	114|	21.99 |	PG-13 |	2006-02-15T05:03:42.000Z	|Trailers,Deleted Scenes	|'administr':9 'alabama':1 'boat':23 'databas':8 'devil':2 'jet':22 'mad':12,18 'must':15 'outgun':16 'panorama':5 'scientist':13,19 'thought':4|
| 10|	ALADDIN CALENDAR|	A Action-Packed Tale of a Man And a Lumberjack who must Reach a Feminist in Ancient China	| 2006	| 1	|	6|	4.99	|63	|24.99	|NC-17 |	2006-02-15T05:03:42.000Z	|Trailers,Deleted Scenes	|'action':5 'action-pack':4 'aladdin':1 'ancient':20 'calendar':2 'china':21 'feminist':18 'lumberjack':13 'man':10 'must':15 'pack':6 'reach':16 'tale':7|
| 


Let's see how many films are present for this table:

```sql
select count(*) from dvd_rentals.film
limit 10
```

Output:

|count|
|-----|
| 1000 |

From the above output it can be observed that there are 1000 films for the film table.


1.4 Film_Category Table
------------------------------------------------------------------------------------------------------------------------
The Film Category Table consists of 3 columns which are film_id, category_id and last_update. Showing the first 10 rows  for this table:

```sql
select * from dvd_rentals.Film_Category
limit 10
```
Output:
 
film_id	| category_id |	last_update|
|-------|--------|------------------|
1 	  |6	   |       2006-02-15T05:07:09.000Z|
2	  |11	   |       2006-02-15T05:07:09.000Z|
3	  |6	   |       2006-02-15T05:07:09.000Z|
4	  |11	   |       2006-02-15T05:07:09.000Z|
5	  |8	   |       2006-02-15T05:07:09.000Z|
6	  |9	   |       2006-02-15T05:07:09.000Z|
7	  |5	   |       2006-02-15T05:07:09.000Z|
8	  |11	   |       2006-02-15T05:07:09.000Z|
9	  |11	   |       2006-02-15T05:07:09.000Z|
10	  |15	   |       2006-02-15T05:07:09.000Z|

1.5 Category Table
------------------------------------------------------------------------------------------------------------------------
The Category Table consists of 3 columns which are category_id, name and last_update. Category_id is the primary column for this table. Showing the first 10 rows  for this table:

```sql
select * from dvd_rentals.category
limit 10
```
Output:
 
category_id	| name |	last_update|
|-------|----------|------------------|
1	    |   Action	   |2006-02-15T04:46:27.000Z
2	    |   Animation  |2006-02-15T04:46:27.000Z
3	    |   Children   |2006-02-15T04:46:27.000Z
4	    |   Classics   |2006-02-15T04:46:27.000Z
5	    |   Comedy	   |2006-02-15T04:46:27.000Z
6	    |   Documentary|2006-02-15T04:46:27.000Z
7	    |   Drama	   |2006-02-15T04:46:27.000Z
8	    |   Family	   |2006-02-15T04:46:27.000Z
9	    |   Foreign	   |2006-02-15T04:46:27.000Z
10	    |   Games	   |2006-02-15T04:46:27.000Z


Lets check how many categories are present for in the category table

``` sql
select COUNT(distinct(category_id)) from dvd_rentals.category
```

Output:

|Count|
|-----|
|16   |


1.6 Film_Actor Table
------------------------------------------------------------------------------------------------------------------------
The Film_actor table is connected to the film table and the actor table. Showing the first 10 rows  for this table:


actor_id    |film_id| last_update
|-----------|-------|------------------|
1		    | 1	    |  2006-02-15T05:05:03.000Z |
1		    | 23	|  2006-02-15T05:05:03.000Z |
1		    | 25	|  2006-02-15T05:05:03.000Z |
1		    | 106	|  2006-02-15T05:05:03.000Z |
1		    | 140	|  2006-02-15T05:05:03.000Z |
1		    | 166	|  2006-02-15T05:05:03.000Z |
1		    | 277	|  2006-02-15T05:05:03.000Z |
1		    | 361	|  2006-02-15T05:05:03.000Z |
1		    | 438	|  2006-02-15T05:05:03.000Z |
1		    | 499	|  2006-02-15T05:05:03.000Z |


This table is connected to the film table and the actor table. A many to many relationship exists between film_id and actor_id.


1.7 Actor Table
------------------------------------------------------------------------------------------------------------------------
The actor table simply shows the first and last name for each actor based off their unique actor_id. we can trace which films a specific actor appears in by joining this table onto the previously discussed film_actor table on the actor_id column. Showing the first 5 rows  for this table:


actor_id |first_name |	last_name  |      last_update
|-------|------------|-------------|---------------|
1	    | PENELOPE   | GUINESS	   | 2006-02-15T04:34:33.000Z |
2	    | NICK	     | WAHLBERG	   | 2006-02-15T04:34:33.000Z |
3	    | ED	     | CHASE	   | 2006-02-15T04:34:33.000Z |
4	    | JENNIFER   | DAVIS	   | 2006-02-15T04:34:33.000Z |
5	    | JOHNNY	 | LOLLOBRIGIDA| 2006-02-15T04:34:33.000Z |



This table is connected to the film table and the actor table. A many to many relationship exists between film_id and actor_id. Lets take a look at the number of actors present in this table:

``` sql
select COUNT(actor_id) from dvd_rentals.actor
```

Output:

|Count|
|-----|
|200  |


This concludes the data exploration. Lets deep dive into the data analysis part to explore the data on deeper level.













