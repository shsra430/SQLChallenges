# Danny's Diner Solution Queries

Built using **mysql**
***
## What is the total amount each customer spent at the restuarant ?
````sql
SELECT DISTINCT
    customer_id, SUM(price) as Amount_Spent
FROM
    sales
        LEFT JOIN
    menu ON sales.product_id = menu.product_id
GROUP BY customer_id;
````
#### Process
- Amount spent by each customer requires two tables to be used: ``sales`` and ``menu`` that are **joined**.
- The **SUM** function is used on the ``price`` attribute to calculate amount spent.
- The **GROUP BY** statement is used to aggregate the results by customer.

#### Result
![image](https://user-images.githubusercontent.com/54994083/158614622-029e0128-1199-49c3-9707-3419ca063f9d.png)
- Customer A has spent $76 making them the most valuable.
***
## How many days has each customer visited the restaurant ?
````sql
SELECT DISTINCT
    customer_id,
    COUNT(DISTINCT order_date) AS Number_of_Days_Visited
FROM
    sales
GROUP BY customer_id
ORDER BY Number_of_Days_Visited DESC;
````
#### Process
- The **COUNT** function is used to get the number of days a customer visited.
- The attribute ``order_date`` is used to get the visitation information. Using **DISTINCT** function helps to aggregate multiple orders placed on a single visit.
#### Result
![image](https://user-images.githubusercontent.com/54994083/158617201-46696335-af04-43b3-8906-77ff504c4c03.png)
- Customer has visited the most number of times i.e 6
***
## What was the first item from the menu purchased by each customer?
````sql
select customer_id, product_name
from(select distinct customer_id, product_name, order_date, 
dense_rank() over( partition by customer_id order by order_date) as ranked
from sales left join menu on sales.product_id = menu.product_id) table1
where ranked=1
group by customer_id,product_name,order_date;
````
#### Process
- Use the window function **DENSE_RANK()** to rank the orders placed by each customer by date.
- Since this requires order information as well as product name, the tables ``sales`` and ``menu`` are joined.
- Subqueries is then used to grab the first product ordered by each customer i.e. those rows with Rank number = 1.
#### Result
![image](https://user-images.githubusercontent.com/54994083/158631849-588d9cda-6513-46d1-a84d-81ece90b7c5e.png)
****
## What is the most purchased item on the menu and how many times was it purchased by all customers?
````sql
select product_name, count(sales.product_id) as times_purchased
from sales left join menu
on sales.product_id = menu.product_id
group by product_name
order by times_purchased desc
limit 1;
````
#### Process
- The **COUNT** function on product_id from ``sales`` table to get the number of times the item was purchased.
- In order to get the name of the product, the tables ``sales`` is joined with ``menu``.
#### Result
![image](https://user-images.githubusercontent.com/54994083/158632061-0ef4a59c-9d96-4dab-9928-5b59bc8496a8.png)
***
## Which item was the most popular for each customer?
````sql
select customer_id, product_name from(select customer_id, product_name, count(sales.product_id) as purchaseCount , 
dense_rank() over( partition by customer_id order by count(sales.product_id) desc) as Ranked
from sales left join menu on sales.product_id = menu.product_id
group by customer_id, product_name) table1
where Ranked=1;
````
#### Process
- A subquery is used to rank the products bought by customers by order of frequency. For this, the **COUNT** function is used on product id attribute.
- The **DENSE_RANK** function is used in the sub-query to rank items so that items with same values get the same rank value, with consecutive rank numbers.
- The main query then extracts customer names from rows where rank value is 1.
#### Result
![image](https://user-images.githubusercontent.com/54994083/158632304-06b495e7-60ea-42ea-8aaf-6449773cc808.png)
- Ramen is the most popular item on the menu
***
## Which item was purchased first by the customer after they became a member?
````sql
select customer_id,order_date,product_name from (select members.customer_id, join_date,product_id, order_date,
dense_rank()over(partition by members.customer_id order by order_date) as ranks
from sales left join members on sales.customer_id= members.customer_id
where order_date >= join_date) table1 left join menu
on table1.product_id = menu.product_id
where ranks=1
group by customer_id;
````
#### Process
- In order to get the ranked list of orders placed by each customer after they became a member. For this the **DENSE_RANK** is used partitioned by customers.
- In the outer query, the customers with rank number=1 are extracted using the ``WHERE`` clause.
#### Result
![image](https://user-images.githubusercontent.com/54994083/158632591-5089d44d-29fc-4408-89ef-fb436c8e94ef.png)
***
## Which item was purchased just before the customer became a member?
````sql
select customer_id, product_name from (SELECT 
    sales.customer_id,
    product_id,
    order_date,
    join_date,
    join_date - order_date AS gap,
    dense_rank() over (partition by customer_id order by order_date desc) as Ranked
FROM
    sales
        LEFT JOIN
    members ON sales.customer_id = members.customer_id
WHERE
    order_date < join_date)table1
left join menu on table1.product_id = menu.product_id
where Ranked=1;
````
#### Process
- Using a subquery, partition by customers and rank them according to the order_date in descending order limiting to only those orders where order_date < join_date. The ``WHERE`` clause is used to manipulate and extract the desired subset of rows.
- The outer query then extracts the customers and the names of products they purchased just before they became members.
#### Result
![image](https://user-images.githubusercontent.com/54994083/158633985-682c812e-a6c8-49fb-bcef-b179290c7aab.png)
***
## What is the total items and amount spent for each member before they became a member?
````sql
SELECT 
    sales.customer_id,
    COUNT(DISTINCT sales.product_id) AS Total_Items,
    SUM(price) AS Amount_Spent
FROM
    sales
        LEFT JOIN
    menu ON sales.product_id = menu.product_id
        LEFT JOIN
    members ON sales.customer_id = members.customer_id
WHERE
    order_date < join_date
GROUP BY sales.customer_id;
````
#### Process
- The **COUNT** function is used here to aggregate the number of different products purchased by each customer. 
- Use the **SUM** function to find the total amount spent by each customer
- The **WHERE** clause is then used to segregate the rows where the orders were made before the joining date of the customer.
#### Result
![image](https://user-images.githubusercontent.com/54994083/159010420-85688825-2495-4468-b358-cc32e66e6071.png)

***
## If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
````sql
SELECT 
    customer_id, SUM(Points) AS Total_Points
FROM
    (SELECT 
        customer_id,
            order_date,
            sales.product_id,
            product_name,
            price,
            IF(product_name <> 'sushi', price * 10, price * 10 * 2) AS Points
    FROM
        sales
    LEFT JOIN menu ON sales.product_id = menu.product_id) table1
GROUP BY customer_id
ORDER BY Total_Points DESC;
````
#### Process
- A subquery is used to first create a new column ``Points`` that calculates the points for every order placed by the customer.
- The conditional **IF** is used to calculate the points according to the order placed.
- To get the price and product name, the ``sales`` table is joined with the ``menu`` table.
- The outer query is then used to aggregate the points using **SUM** function for every customer achieved using the **GROUP BY** clause.
#### Result
![image](https://user-images.githubusercontent.com/54994083/158634218-07b33404-365e-4317-a503-584eda869c83.png)
***
## In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B 
## have at the end of January?
#### Process
#### Result
