#  :avocado: FOODIE-FI
![image](https://user-images.githubusercontent.com/54994083/179741981-b7465522-1bea-4de4-8c2d-924365952f28.png)

## :pencil2: Table of Contents

- [Project Brief](#clipboard-project-brief)
- [Data Set](#file_folder-data-set)
- key Business Questions
- Solutions
  - [Customer Journey](#tv-customer-journey)
  - [Data Analysis](https://github.com/shsra430/SQLChallenges/blob/main/Case%20Study%203%20-%20Foodie%20Fi/Solutions.md#bar_chart-data-analysis)
  - Challenge Extension Question
 
***

#### :clipboard: Project Brief

This is the third case study of Dannys 8weeksqlchallenge. This dataset is from a subscription based startup business **Foodie-Fi** that streams food videos & cooking shows from all over the world. They sell both monthly as well as annual plan of subscriptions to their customers.The overarching aim of this analysis is to answer business questions that can help the business make future decisions on new investments and features and ensure that these decisions are data-driven & leverage the power of business intelligence for growth.
The dataset currently contains two tables: plans & subscriptions.
More information about the tables can be found [here](https://8weeksqlchallenge.com/case-study-3/).

#### :file_folder: Data Set
Below is the image that shows the ER diagram of the tables in the database.

![image](https://user-images.githubusercontent.com/54994083/179738560-c5a813e8-9778-4135-a150-c975ca7fb37e.png)

As mentioned in the project brief [here](https://8weeksqlchallenge.com/case-study-3/) , this case study is split into an initial data understanding question before diving straight into data analysis questions before finishing with 1 single extension challenge.
The first table **plans** contains information about the type of plans offered in Foodie-Fi.

<img width="143" alt="image" src="https://user-images.githubusercontent.com/54994083/179754020-abc52578-00af-4695-9667-d17803785a0b.png">

The company offers 5 different kinds of plans indicated by a whole number between 0 and 4:
  * Trial : Indicated by '0' is a trail period that customers can avail. This lasts for 7 days after which the plan automatically converts to pro monthly.
  * Basic Monthly: Indicated by '1' is a monthly plan that costs $9.90, in which customers can stream unlimited video. This plan is only available monthly.
  * Pro Monthly: Infdicated by '2' is a monthly plan that costs $19.90, in which customers can stream unlimited videos and also have download access of videos for offline viewing.
  * Pro Annual: Indicated by '3' is an annual plan that costs $199. This is the same as plan '2' except it is a 1 year long plan.
  * Churn: Indicated by '4' is to signify customers that have cancelled their subscription.

The second table **subscriptions** is the fact table that contains information about customer subscriptions & journey with Foodie-Fi. 
Customer subscriptions show the exact date where their specific plan_id starts. Here is a look at the first few records from the table.

<img width="149" alt="image" src="https://user-images.githubusercontent.com/54994083/179755782-296aa19a-e387-44dd-91fe-bd7a5e245e5b.png">

If customers downgrade from a pro plan or cancel their subscription - the higher plan will remain in place until the period is over - the start_date in the subscriptions table will reflect the date that the actual plan changes.

When customers upgrade their account from a basic plan to a pro or annual pro plan - the higher plan will take effect straightaway.

When customers churn - they will keep their access until the end of their current billing period but the start_date will be technically the day they decided to cancel their service.

##### :thought_balloon: Key Business Questions
A. Customer Journey
Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.

Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!

B. Data Analysis Questions
  1. How many customers has Foodie-Fi ever had?
  2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
  3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name
  4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
  5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
  6. What is the number and percentage of customer plans after their initial free trial?
  7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
  8. How many customers have upgraded to an annual plan in 2020?
  9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

C. Challenge Payment Question
The Foodie-Fi team wants you to create a new payments table for the year 2020 that includes amounts paid by each customer in the subscriptions table with the following requirements:

  - monthly payments always occur on the same day of month as the original start_date of any monthly paid plan
  - upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
  - upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
  - once a customer churns they will no longer make payments
***
#### A. Customer Journey
From the sample 'subscriptions' table data, it can observed that most customers opt for the 'pro monthly' plan after a trial period.
#### B. Data Analysis
The section [here](https://github.com/shsra430/SQLChallenges/blob/main/Case%20Study%203%20-%20Foodie%20Fi/DataAnalysis.md#avocado-foodie-fi)) contains the solution queries that answer the Data Analysis Questions in part B of this challenge.
#### C. Challenge Extension
