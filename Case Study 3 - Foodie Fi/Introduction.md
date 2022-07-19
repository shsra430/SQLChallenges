#  :avocado: FOODIE-FI
![image](https://user-images.githubusercontent.com/54994083/179741981-b7465522-1bea-4de4-8c2d-924365952f28.png)

## :pencil2: Table of Contents

- [Project Brief](#clipboard-project-brief)
- [Data Set](#file_folder-data-set)
- Solutions
  - [Customer Journey](#tv-customer-journey)
  - [Data Analysis](https://github.com/shsra430/SQLChallenges/blob/main/Case%20Study%203%20-%20Foodie%20Fi/Solutions.md)
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
  
#### :tv: Customer Journey
From the sample 'subscriptions' table data, it can observed that most customers opt for the 'pro monthly' plan after a trial period.
