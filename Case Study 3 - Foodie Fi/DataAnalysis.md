#  :avocado: FOODIE-FI

### :bar_chart: DATA ANALYSIS 
:bulb: This is the second part of the challenge where the following set of business questions need to be answered using SQL. The queru solutions for the questions are built using **MySQL** on MySQL Workbench 8.0.

- SQL level: Intermediate
- Aggregations, joins, Window Functions, CTE, Sub-Queries,Pivots with CASE WHEN, COUNT CASE, DATETIME functions, Typecasting,TEXT & String Manipulation

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

- *Most of the customers opted for the basic monhtly plan after their free trial. But this could also be because basic monthly is the plan that the system defaults to once the trial ends. *
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
#### 7.What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
<img width="188" alt="image" src="https://user-images.githubusercontent.com/54994083/179988335-91deca99-6df9-4782-b632-fddd387bf1a1.png">

- *Most of the active subscriptions are for the pro monthly plan. But, Foodie-Fi needs to work on conversion rates as **23.6% churn** seems high.*
#### :white_check_mark: Process
````sql
with ranked_cte as (
select subscriptions.customer_id,subscriptions.plan_id,subscriptions.start_date,
plans.plan_name,rank() over(partition by subscriptions.customer_id order by subscriptions.start_date desc) as ranks
from subscriptions left join plans
on subscriptions.plan_id=plans.plan_id
where start_date<='2020-12-31')

SELECT 
    plan_name, COUNT(plan_name) AS customercount,
    concat(round((COUNT(plan_name)/(select count(distinct customer_id) from subscriptions))*100,1),"%") as share_of_subscriptions
FROM
    ranked_cte
WHERE
ranks =1
GROUP BY plan_name
order by customercount DESC;
````
- The requirement here is to extract all *active* subscriptions & get the distributions of various plans among them. This analysis also needs to be restricted to only records from the year 2020.
- The solution query for this question uses the window function `RANK()` to extract all the active subscriptions for every customer by ranking the records per customer in the descending order of start_date.
#### 8.How many customers have upgraded to an annual plan in 2020?
<img width="120" alt="image" src="https://user-images.githubusercontent.com/54994083/179989486-cb7c060c-78bc-40af-ae18-74762818d2dc.png">

#### :white_check_mark: Process
````sql
SELECT 
    plan_name, COUNT(DISTINCT customer_id) AS customer_count
FROM
    subscriptions
        LEFT JOIN
    plans ON subscriptions.plan_id = plans.plan_id
WHERE
    subscriptions.plan_id = 3
        AND start_date <= '2020-12-31';
````
#### 9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
<img width="104" alt="image" src="https://user-images.githubusercontent.com/54994083/180000199-f1572afd-d0bc-40fc-9b66-576450e39841.png">

- *It takes a customer an average of 3 months to subscribe to an annual plan*
#### :white_check_mark: Process
````sql
WITH days_to_annual AS (SELECT 
    q.customer_id,
    q.start_date AS annual_start_date,
    MIN(s.start_date) AS trial_starting_date,
    datediff(q.start_date ,MIN(s.start_date)) AS days_needed
FROM
    (SELECT 
        *
    FROM
        subscriptions
    WHERE
        plan_id = 3) q
        LEFT JOIN
    subscriptions s ON q.customer_id = s.customer_id
GROUP BY q.customer_id) SELECT round(AVG(days_needed),0) AS Average_Days_to_Annual_Plan FROM days_to_annual;
````
- The temporary table `days_to_annual` captures the 'start_date' of trial period of all customers & the 'start_date' of all annual plan of all customers who opted for an annual plan. 
- The `DATEDIFF` function is used to extract the difference between the start dates of the two plans in number of days as 'days_needed'.
- The `AVG` function is then used on the 'days_needed' present in the CTE `days_to_annual`
#### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
<img width="159" alt="image" src="https://user-images.githubusercontent.com/54994083/180430418-bfdb5082-5d9a-4551-970b-f982c75099fc.png">

- *Most customers take 1-30 days to upgrade to the annual plan with a very few of them taking more than 270 days or 9 months*
#### :white_check_mark: Process
````sql
WITH days_to_annual_cte AS (SELECT 
    q.customer_id,
    q.start_date AS annual_start_date,
    MIN(s.start_date) AS trial_starting_date,
    datediff(q.start_date ,MIN(s.start_date)) AS days_needed
FROM
    (SELECT 
        *
    FROM
        subscriptions
    WHERE
        plan_id = 3) q
        LEFT JOIN
    subscriptions s ON q.customer_id = s.customer_id
GROUP BY q.customer_id), with_30day_bins_cte AS (SELECT *,floor(days_needed/30) AS assigned_bucket
FROM days_to_annual_cte)SELECT CONCAT((assigned_bucket*30)+1,'-',(assigned_bucket+1)*30,'days') AS Day_period,
COUNT(days_needed) AS Number_of_subscribers,ceil(AVG(days_needed)) as AVG_Days_Needed
FROM with_30day_bins_cte
GROUP BY assigned_bucket
ORDER BY assigned_bucket;
````

- The first CTE in this query `days_to_annual_cte` captures information about the difference in the number of days between trial & annual upgrade for every customer.
- The second CTE `with_30day_bins_cte`, for every customer, the difference in the number of days to upgrade to annual plan is assigned a bin to which it belongs. This way the 'days_needed' column is partitioned into equal interval buckets.
- The outer query selects the 30-day buckets, and counts the number of occurrences or frequency of the number of customers that fall into each of the buckets.
#### 11.How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
<img width="98" alt="image" src="https://user-images.githubusercontent.com/54994083/180435848-76e8a98a-49ec-45b8-a60c-1f78f26e738a.png">

- There were no customers who downgraded from a pro monthly plan to a basic monthly plan in the year 2020.
#### :white_check_mark: Process
````sql
SELECT 
    COUNT(DISTINCT customer_id) as Number_of_Customers
FROM
    (SELECT 
        customer_id,
            CASE
                WHEN plan_id = 1 AND YEAR(start_date = 2020) THEN start_date
            END AS basic_monthly_startdate,
            CASE
                WHEN plan_id = 2 AND YEAR(start_date = 2020) THEN start_date
            END AS pro_monthly_startdate
    FROM
        subscriptions) query2
WHERE
    (basic_monthly_startdate IS NOT NULL)
        AND (pro_monthly_startdate IS NOT NULL)
        AND (pro_monthly_startdate < basic_monthly_startdate);

````
- To get this information, I have used the concept of subqueries for better readability. 
- The subquery aliased `query2`, the query is used to capture the start dates of basic and pro annual plans of all customers, only when the year of both start dates is equal to 2020. The `CASE WHEN` is used to pivot the table.
- The outer query checks for those records that have a non null value in the start dates of both basic and pro monthly plans and the filter to check if pro monthly plan has a start date that is earlier than the start date of the basic monthly plan.
- The SQL `WHERE` clause is used to enforce the above filters.
