# :bike::pizza:Pizza Runner Solution Queries:pizza::bike:
:bulb: Built using MySQL

- SQL level: Intermediate
- Aggregations, joins, Window Functions, CTE, Sub-Queries,Pivots with CASE WHEN, COUNT CASE, DATETIME functions, Typecasting,TEXT & String Manipulation

#### Data Cleaning Steps Involved
##### Table: customer_orders 
1.  On inspection, it was observed that there were null and empty values in the 'extras' and 'exlcusions' columns of the 'customer_orders' table. These will be replaced with a categorical valye 'NA'.
2.  The date and time values from the column 'order_time' have been extracted into separate columns for temporal analysis if required.
3.  A new table is created as a result of the above data manipulation steps
````sql
insert into tempcustomer_orders(order_id,customer_id,pizza_id,exclusions,extras,order_time,orderDate,orderTime)
select order_id,customer_id,pizza_id, case when
exclusions='null' or exclusions='' then 'NA' else exclusions end as exclusions,
case when extras='null' or extras='' or extras is null then 'NA'  else extras end as extras,order_time,
date(order_time),time(order_time) 
from customer_orders;

`````
- null valued and null in pickup_time, distance, duration, cancellation (also has empty fields)
-- remove km and mins, minutes, minute appended to the values in distance and duration columns
-- create a new runner_orders table temprunner_orders with pickuptime as datetime datatype, distance 
-- and duration as float datatype

##### Table: runner_orders 
1. The columns 'pickup_time', 'distance','duration','cancellation' have null values and null entries. These are handled by replacing with 'null' keyword
2. The 'distance' and 'duration' column has inconsistent spellings in units place and this is handled by removing the unit name as we know from schema that all values have the same unit of measurement.
3. The datatype of 'pickuptime' is changed to datatime and distance and duration to float datatype
4. With the above data manipulation steps, a new table is created from the runner_orders table called 'temprunner_orders'
````sql
insert into temprunner_orders(order_id, runner_id,pickup_time,distance, duration, cancellation)
select order_id, runner_id, 
case when pickup_time='null' then NULL else pickup_time end as pickup_time,
case when distance like '%km' then trim('km' from distance)
when distance ='null' then null
else distance end as distance,
case when duration ='null' then NULL
when duration like '%minutes' then trim('minutes' from duration)
when duration like '%mins' then trim('mins' from duration)
when duration like '%minute' then trim('minute' from duration)
else duration end as duration,
case when cancellation='' then 'NA'
when cancellation is null then 'NA'
when cancellation='null' then 'NA'
else cancellation end as cancellation
from runner_orders;
````

##### Table: temprunner_orders 
1.  Remove leading and trailing spaces from the duration and distance columns.
````sql
update temprunner_orders set distance = TRIM(distance);
update temprunner_orders set duration = TRIM(duration);
````
With this data cleaning process is completed & now we can move on with answering the key business questions with queries.
The first set of questions are about **Pizza Metrics**
##### 1. How many pizzas were ordered ?
There are a total of 14 :pizza: orders.
##### :white_check_mark:Process 
````sql
SELECT 
    COUNT(*) AS Total_Pizza_Orders
FROM
    tempcustomer_orders;
````
- For this question, the table 'customer_orders' will be required to count the number of orders. The **SQL COUNT()** Statement will be used along with * to count all rows in the customer_orders table.
##### 2. How many unique customer orders were made ?
A total of 5 unique customers have placed various orders.
#### :white_check_mark:Process
````sql
SELECT 
    COUNT(DISTINCT order_id) AS Unique_Orders
FROM
    tempcustomer_orders;
SELECT 
    COUNT(DISTINCT customer_id) AS Unique_Customer_Orders
FROM
    tempcustomer_orders;
````
- To answer this question the COUNT() statement is used again. In order to get the unique customers who placed the orders, the customer_id is counted and to get the number of unique orders that were placed the 'order_id' is counted.
##### 3. How many successful orders were delivered by each runner ?
````
Runner Successful_Orders
1      4
2      3
3      1
````
##### :white_check_mark:Process
````SQL
SELECT 
	runner_id as Runner,
    COUNT(DISTINCT order_id) AS Successful_Orders
FROM
    temprunner_orders
WHERE
    cancellation = 'NA'
GROUP BY Runner;
````
- To get the count of successful orders, COUNT() is used to aggregate the distinct orders, segragated by runners using GROUP BY clause on runner_id. 
- Since we are only interested in successful orders, cancelled orders are excluded by filtering using the where clause, to get only those orders in which cancellation
value is NA. 
##### 4. How many of each type of pizza was delivered ?
````
PizzaType   DeliveryCount
Meatlovers    9
Vegetarian    3
````
##### :white_check_mark:Process
````sql
SELECT 
    pizza_name AS PizzaType,
    COUNT(tempcustomer_orders.order_id) AS DeliveryCount
FROM
    tempcustomer_orders
        LEFT JOIN
    temprunner_orders ON tempcustomer_orders.order_id = temprunner_orders.order_id
        LEFT JOIN
    pizza_names ON tempcustomer_orders.pizza_id = pizza_names.pizza_id
WHERE
    cancellation = 'NA'
GROUP BY PizzaType;
````
- Since this requires information from 3 different tables, the data is extracted using left join. However, the question is only interested in those orders that were delivered. 
- Therefore, we need to restrict our results to only those rows where cancellation is NA or distance > 0.This filter is acheived using a WHERE clause 
and the aggregation is done by PizzaType using GROUP BY.
##### 5. How many Vegetarian and MeatLovers were ordered by each customer ?
<img width="214" alt="image" src="https://user-images.githubusercontent.com/54994083/179524647-5c2d81b9-e2ac-4f18-9d13-b1dc543659e8.png">

##### :white_check_mark:Process
````sql
SELECT 
    customer_id AS Customer,
    COUNT(CASE
        WHEN pizza_name = 'Vegetarian' THEN 1
    END) AS 'Vegetarian Pizza(s)',
    COUNT(CASE
        WHEN pizza_name = 'MeatLovers' THEN 1
    END) AS 'MeatLovers Pizza(s)'
FROM
    tempcustomer_orders
        LEFT JOIN
    pizza_names ON tempcustomer_orders.pizza_id = pizza_names.pizza_id
GROUP BY customer_id;
````
- For this question, the data needs to be segmented with two different factors: Pizza Type and Customer ID. 
- It also requires information from two different tables 'tempcsutomer_orders' for orders information & 'pizza_names' for the type of pizza. They are joined by 'pizza_id' column. 
- To get the count of each type of pizza, COUNT CASE() is used to filter orders and then apply COUNT aggregation. 
- And to get the counts per customer, the GROUP BY is applied to 'customer_id'.
##### 6. What was the maximum number of pizzas delivered in a single order ?
````
order_id    pizzacount
4             3
````
##### :white_check_mark:Process
````sql
SELECT 
    tempcustomer_orders.order_id, COUNT(pizza_id) AS pizzacount
FROM
    tempcustomer_orders
        LEFT JOIN
    temprunner_orders ON tempcustomer_orders.order_id = temprunner_orders.order_id
WHERE
    cancellation = 'NA'
GROUP BY 1
ORDER BY pizzacount DESC
LIMIT 1;
````
- Since we need to focus on delivered orders, we need only those records where cancellation is NA. To answer this question, we make use of two tables 'tempcustomer_orders' & 'temprunner_orders'. 
- For every order, we need to count the number of pizza_id(s), order these in descending order of count & get the record with the highest count. 
- This is achieved by using GROUP BY on order_id, ORDER BY count of pizza_id and then use the LIMIT keyword to control the number of records retrieved, in this case 1.
##### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
<img width="203" alt="image" src="https://user-images.githubusercontent.com/54994083/179529485-c54ba064-3fe4-43b7-8408-83ecbb820b0c.png">

##### :white_check_mark:Process
````sql
SELECT 
    customer_id,
    SUM(CASE
        WHEN exclusions <> 'NA' OR extras <> 'NA' THEN 1
        ELSE 0
    END) AS changed_orders,
    SUM(CASE
        WHEN exclusions = 'NA' AND extras = 'NA' THEN 1
        ELSE 0
    END) AS nochange_orders
FROM
    tempcustomer_orders
        LEFT JOIN
    temprunner_orders ON tempcustomer_orders.order_id = temprunner_orders.order_id
WHERE
    cancellation = 'NA'
GROUP BY customer_id;
````
- To get the number of orders with changes done, the records where 'exclusions' and 'extras' are not null are to be counted & to get the number of orders without
changes, those records with 'exclusions' & 'extras' being NULL have to be counted. For this the SUM aggregation is used with CASE WHEN.
- Since we need this information per customer ID , the GROUP BY is 
applied on customer_id. 
##### 8. How many pizzas were delivered that had both exclusions and extras ?
````
Countof
1
````
##### :white_check_mark:Process
````sql
SELECT 
    SUM(CASE
        WHEN extras <> 'NA' AND exclusions <> 'NA' THEN 1
    END) AS Countof
FROM
    tempcustomer_orders
        LEFT JOIN
    temprunner_orders ON tempcustomer_orders.order_id = temprunner_orders.order_id
WHERE
    cancellation = 'NA';
````
- For this question, the records need to have a value in both 'exclusions' and 'extras' columns of the tempcustomer_orders. But, it is also important to filter and apply this logic toonly records where cancellation column in temprunner_orders is NA. 
- Therefore the solution query used SUM aggregation with CASE WHEN toprocess only the interested records with a WHERE clause filter on 'cancellation' column.
##### 9. What was the total volume of pizzas ordered for each hour of the day ?
<img width="105" alt="image" src="https://user-images.githubusercontent.com/54994083/179532547-d366be09-924d-42c2-b60e-3b987fb81316.png">

- *Evenings until late night seems to be busier overall, along with an afternoon rush.*
##### :white_check_mark:Process
````sql
SELECT 
    HOUR(orderTime) AS Dayhour, COUNT(order_id) AS Order_Volume
FROM
    tempcustomer_orders
GROUP BY Dayhour;
````
- For this question, there is only one table needed: tempcustomer_orders. To get the hour information, the HOUR function is used on 'orderTime' column and 
COUNT aggregation is used on 'order_id'. 
- Since we need the number of orders *per hour* , GROUP BY is used on each HOUR of the orderTime.

##### 10. What was the volume of orders for each day of the week ?
<img width="119" alt="image" src="https://user-images.githubusercontent.com/54994083/179533609-ab85765f-2c80-48bd-9114-03c6a4fe68a0.png">

- *Midweek & Saturday seem to be quite popular days of business.
##### :white_check_mark:Process
````sql
SELECT 
    DAYNAME(orderDate) AS OrderDay,
    COUNT(order_id) AS Order_Volume
FROM
    tempcustomer_orders
GROUP BY OrderDay;
````
***
The next set of business questions are about **runner and customer experience**.
##### 1. How many runners signed up for each 1 week period ? ( i.e week starts 2021-01-01 )
<img width="184" alt="image" src="https://user-images.githubusercontent.com/54994083/179536280-8371120c-3bca-4f54-a612-7d9ddb8ea452.png">

- *The first week has seen the maximum number of sign ups.*
#### Process
````sql
SELECT 
    WEEK(registration_date), COUNT(runner_id)
FROM
    runners
GROUP BY 1;
````

##### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
<img width="115" alt="image" src="https://user-images.githubusercontent.com/54994083/179537145-a8e20430-6c74-4f04-89c9-b5b4018a194e.png">

- *It looks like Runner 2 has the highest avg time difference between order time & pick up time, with runner 3 being the most efficient.*
##### :white_check_mark:Process
````sql
with pickuptimes as (
select runner_id, TIMESTAMPDIFF(MINUTE,tempcustomer_orders.order_time,pickup_time) as timeDifference
from tempcustomer_orders left join temprunner_orders
on tempcustomer_orders.order_id = temprunner_orders.order_id
)select runner_id as Runner, avg(timeDifference) as Avg_Timeto_HQ from pickuptimes
group by runner_id;
````
- To answer this business question, I have created a CTE with difference between order time and pick up times in minutes for every runner.
- This CTE is then used in a subsequent query with AVG aggregation function on time difference, with GROUP BY on runner_id to analyse the records runner-wise.
- 
##### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
<img width="195" alt="image" src="https://user-images.githubusercontent.com/54994083/179538383-7b47a74b-8f3a-4ffa-856f-13181126af03.png">

- *It looks like with the increase in the number of pizzas per order, the average difference between order time & pick up time also increases.*
##### :white_check_mark:Process
````sql
with timeinfo as(
select tempcustomer_orders.order_id, count(pizza_id) as number_of_pizzas, order_time as lastordertime,
pickup_time as lastpickuptime 
from tempcustomer_orders left join temprunner_orders on tempcustomer_orders.order_id = temprunner_orders.order_id
group by tempcustomer_orders.order_id 
)select number_of_pizzas as Countof_Pizzas_Ordered, AVG(TIMESTAMPDIFF(MINUTE,lastordertime,lastpickuptime)) as Avg_Time_to_Prepare
from timeinfo
group by number_of_pizzas
order by Avg_Time_to_Prepare desc;
````
- For this question, I again used a CTE. It makes querying more efficient & helps to structurally align the steps needed
to arrive at the final solution query. The CTE is used to create a table that contains the count of pizzas per order, the order time and finally the pick up time.
- From the CTE, the next query is used to GROUP BY number of pizzas per order, and using AVG aggregation on TIMESTAMPDIFF between the pickup time & order time
to get the average time to prepare value. This ofcourse ignores any other causes for delay in pick up times.
- Finally, the records are ordered in desc of count of pizzas per order using ORDER BY with 'desc' keyword.

##### 4. What was the average distance travelled for each customer ?
<img width="148" alt="image" src="https://user-images.githubusercontent.com/54994083/179541820-bd2ade55-c970-4c8c-91f3-41f8bae55d78.png">

- *Looks like customer 105 is located the farthest from Danny's house or Pizza Runne HQ*
##### :white_check_mark:Process
````sql
SELECT 
    customer_id AS Customer,
    ROUND(AVG(distance), 2) AS Avg_Distance_Traveled
FROM
    tempcustomer_orders
        LEFT JOIN
    temprunner_orders ON tempcustomer_orders.order_id = temprunner_orders.order_id
GROUP BY customer_id;
````
##### 5. What was the difference between the longest and shorted delivery times for all orders ?
<img width="243" alt="image" src="https://user-images.githubusercontent.com/54994083/179548760-b588f4f9-36c2-41b2-8bde-cb7d41e03e81.png">

- *There is a 30 minute difference between the highest and lowest delivery times.*
##### Process
````sql
SELECT 
    MAX(duration),
    MIN(duration),
    MAX(duration) - MIN(duration) difference_in_delivery_times
FROM
    (SELECT 
        *
    FROM
        temprunner_orders
    WHERE
        duration IS NOT NULL) table1;
````
- To answer this question, I have made use of a subquery. The subquery table1 contains all records in temprunner_orders where duration is not null. The main query uses table1 to get the final result. 
- The aggregation functions MAX, MIN are used to find the highest and lowest duration values in the dataset and then arithmetic subtraction is used to find the difference between them.

##### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
<img width="472" alt="image" src="https://user-images.githubusercontent.com/54994083/179550092-572e7692-11b7-4367-ae45-58f3bcd2811b.png">

- *It can be seen that for orders that come later in the day from late afternoon onwards, the speed of runners increase. The cause for this can be found out with further analysis of order volume, day of week etc*
##### :white_check_mark:Process
````sql
SELECT 
    *, ROUND(distance/ (duration/60), 2) AS order_speed_km_per_hour
FROM
    temprunner_orders
order by runner_id,order_speed_km_per_hour;
````
##### 7. What is the successful delivery percentage for each runner?
<img width="307" alt="image" src="https://user-images.githubusercontent.com/54994083/179551928-c16c83d7-e363-49e9-ab58-9a284bd1b11f.png">

- *Runner 1 has been the most successful in terms of delivery percentage as most of his orders convert into deliveries. Whereas, runner 3 has the lowest successful delivery precentage with only 1 out of 2 of his orders getting delivered.*
##### :white_check_mark:Process
````sql
SELECT 
    runner_id,
    COUNT(DISTINCT order_id) AS total_orders,
    COUNT(CASE
        WHEN cancellation = 'NA' THEN order_id
    END) AS delivered_orders,
    ROUND(COUNT(CASE
                WHEN cancellation = 'NA' THEN order_id
            END) / COUNT(DISTINCT order_id) * 100,
            0) AS successful_delivery_percentage
FROM
    temprunner_orders
GROUP BY runner_id;
````
***
The following section contains queries to solve business questions on **Ingredient Optimization.**
##### 1. What are the standard ingredients for each pizza ?  
<img width="316" alt="image" src="https://user-images.githubusercontent.com/54994083/179606280-aad6ebe5-0e65-4630-b8e2-bed1efd229ae.png">

-  *Cheese & Muschroom are two ingredients that are common to both Meat & Vegetarian pizzas at Danny's House.*
##### :white_check_mark:Process
````sql
with cte1 as 
(select query1.pizza_id,topping_name from (SELECT
  pizza_id,
  SUBSTRING_INDEX(SUBSTRING_INDEX(toppings, ',', numbers.n), ',', -1) toppings
FROM
  (SELECT 1 n UNION ALL SELECT 2 
   UNION ALL SELECT 3 UNION ALL SELECT 4 union all 
   select 5 union all select 6
   union all select 7 union all select 8 union all select 9 
   union all select 10 union all select 11 union all select 12) numbers INNER JOIN pizza_recipes
  ON CHAR_LENGTH(toppings)-CHAR_LENGTH(REPLACE(toppings, ',', ''))>=numbers.n-1)query1
  left join pizza_toppings on query1.toppings= pizza_toppings.topping_id)
  select pizza_names.pizza_name as Pizza, group_concat(distinct topping_name) as `Standard Toppings` from cte1
left join pizza_recipes
on cte1.pizza_id = pizza_recipes.pizza_id
left join pizza_names on cte1.pizza_id= pizza_names.pizza_id
group by cte1.pizza_id;
````
- The solution query for makes use of CTE as well as a subquery. The innermost query is used to convert comma separated ingredient list of toppings to long form. 
- This is then joined with pizza_toppings to get the names of the toppings. Using the CTE, the final query joins with pizza_recipes & pizza_names to get the pizza name.
- The GROUP_CONCAT() is used to re-group toppings into a comma separated list for each of the two pizza types. 

#####  2. What was the most common exclusion ?
<img width="167" alt="image" src="https://user-images.githubusercontent.com/54994083/179616299-53262146-206c-410d-84e8-30320593fa24.png">

- *Cheese seems to be the ingredient that is most commonly excluded and it is to be noted that it is one of the ingredients that is common to both pizza types. It may be a good idea for the business to include a **lactose-free** option on the menu which will make the process of ordering more straightforward.*
##### Process
````sql
SELECT 
    topping_name AS Ingredient,
    Frequency AS Number_of_times_excluded
FROM
    (SELECT 
        exclusions, COUNT(*) AS Frequency
    FROM
        tempcustomer_orders_exclusions
    GROUP BY exclusions) query2
        LEFT JOIN
    pizza_toppings ON query2.exclusions = pizza_toppings.topping_id
WHERE
    topping_name <> ''
ORDER BY Frequency DESC;
````

##### 4. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients. For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"

<img width="368" alt="image" src="https://user-images.githubusercontent.com/54994083/179629277-68a7da09-ffe0-453f-86e9-a33ce316c0d5.png">

##### Process
To answer this question, I had to create several temporary tables. I am going to try and cover them all here. However, I also strongly feel like there is definitely a simpler way to solve it. I will be happy to learn it. Feedbacks are welcome!
````sql
create temporary table pizza_recipes_extended
SELECT 
    pizza_id,
    SUBSTRING_INDEX(SUBSTRING_INDEX(toppings, ',', numbers.n),
            ',',
            - 1) toppings
FROM
    (SELECT 1 n UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8) numbers
        INNER JOIN
    pizza_recipes ON CHAR_LENGTH(toppings) - CHAR_LENGTH(REPLACE(toppings, ',', '')) >= numbers.n - 1
ORDER BY pizza_id;
````
````sql
create temporary table pizza_recipes_toppingname
SELECT 
    pizza_id, topping_name
FROM
    pizza_recipes_extended
        LEFT JOIN
    pizza_toppings ON pizza_recipes_extended.toppings = pizza_toppings.topping_id;

````
````sql
create temporary table tempcustomer_orders_extended_exclusions_and_extras
select order_id,customer_id,pizza_id,exclusions,order_time,orderDate,orderTime, SUBSTRING_INDEX(SUBSTRING_INDEX(extras, ',', numbers.n), ',', -1) extras
FROM
(SELECT 1 n UNION ALL SELECT 2 ) numbers INNER JOIN tempcustomer_orders_extended_exclusions
ON CHAR_LENGTH(extras)-CHAR_LENGTH(REPLACE(extras, ',', ''))>=numbers.n-1;
````
````sql
create temporary table final_customer_orders_table_one
select 
query3.pizza_id,
query3.order_id,
query3.customer_id,
query3.exclusions,
query3.order_time,
query3.orderDate,
query3.orderTime,
query3.extras,
query3.topping_name,
pizza_toppings.topping_name as exclusions_name
from (
select 
tempcustomer_orders_extended_exclusions_and_extras.pizza_id,
tempcustomer_orders_extended_exclusions_and_extras.order_id,
tempcustomer_orders_extended_exclusions_and_extras.customer_id,
tempcustomer_orders_extended_exclusions_and_extras.exclusions,
tempcustomer_orders_extended_exclusions_and_extras.order_time,
tempcustomer_orders_extended_exclusions_and_extras.orderDate,
tempcustomer_orders_extended_exclusions_and_extras.orderTime,
tempcustomer_orders_extended_exclusions_and_extras.extras,
pizza_recipes_toppingname.topping_name
from tempcustomer_orders_extended_exclusions_and_extras left join pizza_recipes_toppingname
on tempcustomer_orders_extended_exclusions_and_extras.pizza_id = pizza_recipes_toppingname.pizza_id
)query3
left join pizza_toppings on query3.exclusions = pizza_toppings.topping_id ;
````
````sql
create temporary table final_customerorders
select 
query4.pizza_id,
query4.order_id,
query4.customer_id,
query4.exclusions,
query4.order_time,
query4.orderDate,
query4.orderTime,
query4.extras,
query4.topping_name,
query4.exclusions_name,
pizza_toppings.topping_name as extras_name
from (
select final_customer_orders_table_one.pizza_id,
final_customer_orders_table_one.order_id,
final_customer_orders_table_one.customer_id,
final_customer_orders_table_one.exclusions,
final_customer_orders_table_one.order_time,
final_customer_orders_table_one.orderDate,
final_customer_orders_table_one.orderTime,
final_customer_orders_table_one.extras,
final_customer_orders_table_one.exclusions_name,
pizza_recipes_toppingname.topping_name
from final_customer_orders_table_one left join pizza_recipes_toppingname
on final_customer_orders_table_one.pizza_id = pizza_recipes_toppingname.pizza_id
)query4
left join pizza_toppings on query4.extras = pizza_toppings.topping_id ;

````
````sql
create temporary table order_ingredient_csv
SELECT 
    order_id,
    pizza_id,
    exclusions_name,
    extras_name,
    GROUP_CONCAT(DISTINCT topping_name) AS ingredients
FROM
    final_customerorders
GROUP BY order_id, pizza_id;
````
Here is the final query which helped to achieve the final output.
Re-attaching the query response for a quicker reference:
<img width="368" alt="image" src="https://user-images.githubusercontent.com/54994083/179629277-68a7da09-ffe0-453f-86e9-a33ce316c0d5.png">

I would like to add that, finally there are a couple of things that need to be worked on in this solution, which I have come to realise later. 
1. In the final result, the names of excluded ingredients have not been removed from the comma separated list. They have only been replaced with a space i.e. ''.
2. There is a case where an ingredient is mentioned in the extras that is not part of the pizza ingredient list. This needs to be exclusively added to the main ingredient list. 

````sql
SELECT 
   query5.order_id, pizza_name,
    CASE
        WHEN
            FIND_IN_SET(extras_name, all_ingredients) >= 1
        THEN
            REPLACE(all_ingredients,
                extras_name,
                CONCAT('2x', extras_name))
        ELSE all_ingredients
    END AS Ingredients_Final
FROM
    (SELECT 
        *,
            CASE
                WHEN FIND_IN_SET(exclusions_name, ingredients) >= 1 THEN REPLACE(ingredients, exclusions_name, '')
                ELSE ingredients
            END AS all_ingredients
    FROM
        order_ingredient_csv) query5 left join pizza_names
        on query5.pizza_id= pizza_names.pizza_id;
````
