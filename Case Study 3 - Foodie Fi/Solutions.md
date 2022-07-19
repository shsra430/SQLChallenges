#  :avocado: FOODIE-FI

### :bar_chart: DATA ANALYSIS 
:bulb: This is the second part of the challenge where the following set of business questions need to be answered using SQL. The queru solutions for the questions are built using **MySQL** on MySQL Workbench 8.0.

***
#### 1. How many customers has Foodie-Fi ever had?
<img width="74" alt="image" src="https://user-images.githubusercontent.com/54994083/179760248-3858426b-e8ad-42dd-a373-c4e5676d120d.png">

- *There are a total of **1000** customers who have been part of the Foodie-Fi customer base.*
##### :white_check_mark: Process
````sql
SELECT 
    COUNT(DISTINCT customer_id) AS TotalCustomers
FROM
    subscriptions;
````
- The `COUNT()` statement is used along with distinct to get the total number of customers on the attribute 'customer_id'.
#### 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
<img width="149" alt="image" src="https://user-images.githubusercontent.com/54994083/179763752-8c89ab22-613f-46c0-b873-4b1e7e9277e0.png">

- There is not particular pattern apparent in the distribution at this point. Further analysis will be required.
- *Most trials was recorded in March followed by January, March, August & Septemeber. February has least number of trial plan subscriptions.*
#### :white_check_mark: Process
````sql
SELECT 
    YEAR(start_date) AS Year,
    MONTHNAME(start_date) AS Month,
    COUNT(*) AS Number_of_Trial_Plans
FROM
    subscriptions
WHERE
    plan_id = 0
GROUP BY MONTH(start_date)
ORDER BY MONTH(start_date) ASC;
````
- `MONTH_NAME` is used to extract name of the month from the 'start_date' field.
- For the group by, `YEAR()` has also been used. This is done to ensure that as the database grows with records from subsequent years, the group criteria continues to fit. But, it can be ignored right now since the data is only limited to a single year 2020.

#### 3.What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name
 <img width="144" alt="image" src="https://user-images.githubusercontent.com/54994083/179766392-be04dcd1-b912-4175-932d-521a6173e567.png">

- *In 2021, most of the events have been **churn** followed by **pro annual** * 
#### :white_check_mark: Process
````sql
SELECT 
    p.plan_name, s.plan_id, COUNT(*) AS Countof_Events
FROM
    subscriptions s
        LEFT JOIN
    plans p ON s.plan_id = p.plan_id
WHERE
    YEAR(s.start_date) > 2020
GROUP BY plan_name
ORDER BY Countof_Events DESC;
````
- Since the question asks for plan name, we need to join the 'subscriptions' table with the 'plans' table. And count all the records for every plan that have a start date after 2020. 
- `GROUP BY` is used on plan name to segment the data & the date filter is applied using a `WHERE` clause.
#### 4.What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
<img width="127" alt="image" src="https://user-images.githubusercontent.com/54994083/179836279-7efabdce-6b58-445e-adfd-c7ca979398e1.png">

- *Overall churn percentage is **30.7%**. A total of 307 customers cancelled their subscriptions in 2020 & 2021*
#### :white_check_mark: Process
````sql
SELECT 
    COUNT(DISTINCT customer_id) AS TotalCustomers,
    ROUND(((COUNT(DISTINCT CASE
                    WHEN plan_id = 4 THEN customer_id
                END) / COUNT(DISTINCT customer_id)) * 100),
            1) AS Churn_percentage
FROM
    subscriptions;
````
- Since we only need to count those records that correspond to churn, `COUNT CASE` has been used along with distinct to count those records where plan_id =4.
- This is then divided by total number of customers. The `ROUND()` is used to round of the decimal value to 2 places. 
#### 5.How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number? 
<img width="314" alt="image" src="https://user-images.githubusercontent.com/54994083/179843027-64b8c61e-d836-4883-acdf-593613716e56.png">

- *Out of the 30.7% customers that have churned, 9% of them have cancelled immediately after their trial period. *
#### :white_check_mark: Process
````sql
select count(distinct customer_id) as Total,
select count(distinct customer_id) as Total,
count(distinct case when plan_id=4 and ranks_assigned=2 then customer_id end) as count_of_immediatelychurned_customers,
concat(round((count(distinct case when plan_id=4 and ranks_assigned=2 then customer_id end)/count(distinct customer_id))*100,0),"%") as percentage_of_immediatelychurned_customers
from 
(select *,
rank() over(partition by customer_id order by start_date asc) as ranks_assigned
from subscriptions)query1;
````
- To get this data point, we need to group records by customer & rank them in ascending order start_date.
- From this ranked dataset, we can extract all records where plan id corresponds to churn action & has the rank 2 which means they canceled just after trial period.
- To rank the records, the window function `RANK()` is used partitioned by customer_id field. 
#### 6. What is the number and percentage of customer plans after their initial free trial?
<img width="184" alt="image" src="https://user-images.githubusercontent.com/54994083/179848469-c0d52195-d3b2-487a-a15b-7750357eedad.png">

- * *
#### :white_check_mark: Process

````sql
with rankedsubscriptions as(select *,
rank() over(partition by customer_id order by start_date asc) as ranks_assigned
from subscriptions)
SELECT 
    p.plan_name AS Plan,
    COUNT(DISTINCT s.customer_id) AS Customer_Count,
    CONCAT(ROUND((COUNT(DISTINCT s.customer_id) / (SELECT 
                            COUNT(DISTINCT customer_id)
                        FROM
                            subscriptions)) * 100,
                    1),
            '%') AS shareof_customers
FROM
    rankedsubscriptions s
        LEFT JOIN
    plans p ON s.plan_id = p.plan_id
WHERE s.plan_id != 0 and ranks_assigned=2
GROUP BY s.plan_id
ORDER BY Customer_Count desc;
````

#### :white_check_mark: Process
#### :white_check_mark: Process
#### :white_check_mark: Process
#### :white_check_mark: Process
