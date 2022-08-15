### :euro: D. CHALLENGE PAYMENT QUESTION

This challenge asks for the creation of a new table `payments` for the year 2020 that includes the revenue brought in from each customer in the subscriptions table with a certain set of conditions. The following are the conditions:

- monthly payments always occur on the same day of month as the original start_date of any monthly paid plan
- upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
- upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
- once a customer churns they will no longer make payments

````sql
CREATE TABLE cte_allsubscriptions(
SELECT s.customer_id, s.plan_id, s.start_date,
p.plan_name, p.price ,
rank() over(partition by s.customer_id order by s.start_date) as payment_order,
lag(p.plan_name,1,'NULL') over (partition by s.customer_id order by s.start_date) as prev_plan
FROM subscriptions s LEFT JOIN plans p on 
s.plan_id=p.plan_id
WHERE year(s.start_date)=2020 and s.plan_id>0
ORDER BY s.customer_id);
````

The above temporary table is used to the initial table which only contains data in which the start of the plan is in the year 2020 and where trial subscriptions are excluded since those records do not bring in any revenue. The table also contains two new columns `payment_order` and `prev_plan` which will be used in the final query. 
In this table the customers are grouped by their ids and ranked in the ascending order of the plan start dates. This rank value is captured in the column `payment_order`. The window function `RANK` is used for this.
The column `prev_plan` has the name of the previous plan for every customer. This column is created using the SQL window function `LAG`. 
Using the above temporary table, I created the main solution query shown below:

````sql
SELECT 
    customer_id,
    plan_id,
    plan_name,
    CASE
        WHEN plan_name = 'basic monthly' THEN start_date
        WHEN
            plan_name = 'pro monthly'
                AND prev_plan = 'basic monthly'
        THEN
            start_date
        WHEN
            plan_name = 'pro annual'
                AND prev_plan = 'basic monthly'
        THEN
            start_date
        WHEN
            plan_name = 'pro annual'
                AND prev_plan = 'NULL'
        THEN
            LAST_DAY(start_date)
        WHEN
            plan_name = 'pro monthly'
                AND prev_plan = 'NULL'
        THEN
            LAST_DAY(start_date)
        WHEN
            plan_name = 'pro annual'
                AND prev_plan = 'pro monthly'
        THEN
            LAST_DAY(start_date)
        ELSE start_date
    END AS payment_start_date,
    CASE
        WHEN plan_name = 'churn' THEN 0
        WHEN
            plan_name = 'pro monthly'
                AND prev_plan = 'basic monthly'
        THEN
            (19.9 - 9.90)
        WHEN
            plan_name = 'pro annual'
                AND prev_plan = 'basic monthly'
        THEN
            (199 - 9.90)
        ELSE price
    END AS price_paid,
    payment_order
FROM
    cte_allsubscriptions;
````

The SQL `CASE WHEN` is used to conditionally create the `payment_start_date` and `price_paid` columns. However, I think that this solution can be more elegantly built, without hard coding the `price_paid` column. I am happy to hear your thoughts and/or feedback on this. 

A screenshot of the result of this query is attached below:

<img width="293" alt="image" src="https://user-images.githubusercontent.com/54994083/184718833-dc7aea7c-e16f-4e74-b2b2-6cc3046047e7.png">
