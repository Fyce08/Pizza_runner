
# **Pizza Runner**



##  **Table Of Contents**
  ### -[Problem Statement](#problem-statement)
  ### -[Dataset](#dataset)
  ### -[Data Preprocessing](#️-data-preprocessing )
  ### -[Questions](#-Questions)
  ### -[Solutions](#-solutions)
  ### -[Recommendations](#Recommendations)


## **Problem Statement**

As John scrolled through his Instagram feed, something instantly grabbed his attention: “80s Retro Styling and Pizza Is The Future!” The vibrant, nostalgic theme resonated with him, sparking a wave of inspiration.

John was hooked on the idea, but he knew that retro-themed pizza alone wouldn’t be enough to secure the seed funding he needed to expand his budding pizza venture. He realized he needed something more—a twist that would set his business apart. That’s when a lightbulb went off: why not take the convenience of ride-sharing and apply it to pizza delivery? And just like that, Pizza Runner was born.

John began by assembling a team of “runners,” dedicated individuals who would deliver hot, fresh pizza straight from Pizza Runner Headquarters to customers’ doors. To bring his vision to life, he took a bold step and maxed out his credit card to hire freelance developers. Their task? To build a sleek mobile app that would make ordering pizza as effortless as hailing a ride.

---

## **Dataset**
John has shared 6 key datasets:

1. ### `runners`

The runners table shows the `registration_date` for each new runner.


|runner_id|registration_date|
|---------|-----------------|
|1        |1/1/2021         |
|2        |1/3/2021         |
|3        |1/8/2021         |
|4        |1/15/2021        |



2. ### `customer_orders`

Customer pizza orders are captured in the `customer_orders` table with 1 row for each individual pizza that is part of the order.

|order_id|customer_id|pizza_id|exclusions|extras|order_time        |
|--------|---------|--------|----------|------|------------------|
|1  |101      |1       |          |      |44197.75349537037 |
|2  |101      |1       |          |      |44197.79226851852 |
|3  |102      |1       |          |      |44198.9940162037  |
|3  |102      |2       |          |null |44198.9940162037  |
|4  |103      |1       |4         |      |44200.558171296296|
|4  |103      |1       |4         |      |44200.558171296296|
|4  |103      |2       |4         |      |44200.558171296296|
|5  |104      |1       |null      |1     |44204.87533564815 |
|6  |101      |2       |null      |null  |44204.877233796295|
|7  |105      |2       |null      |1     |44204.88922453704 |
|8  |102      |1       |null      |null  |44205.99621527778 |
|9  |103      |1       |4         |1, 5  |44206.47429398148 |
|10 |104      |1       |null      |null  |44207.77417824074 |
|10 |104      |1       |2, 6      |1, 4  |44207.77417824074 |



3. ### `runner_orders`

After each orders are received through the system - they are assigned to a runner - however not all orders are fully completed and can be cancelled by the restaurant or the customer.

The `pickup_time` is the timestamp at which the runner arrives at the Pizza Runner headquarters to pick up the freshly cooked pizzas. 

The `distance` and `duration` fields are related to how far and long the runner had to travel to deliver the order to the respective customer.



|order_id|runner_id|pickup_time|distance  |duration|cancellation      |
|--------|---------|-----------|----------|--------|------------------|
|1       |1        |1/1/2021 18:15|20km      |32 minutes|                  |
|2       |1        |1/1/2021 19:10|20km      |27 minutes|                  |
|3       |1        |1/3/2021 0:12|13.4km    |20 mins |*null*             |
|4       |2        |1/4/2021 13:53|23.4      |40      |*null*             |
|5       |3        |1/8/2021 21:10|10        |15      |*null*             |
|6       |3        |null       |null      |null    |Restaurant Cancellation|
|7       |2        |1/8/2020 21:30|25km      |25mins  |null              |
|8       |2        |1/10/2020 0:15|23.4 km   |15 minute|null              |
|9       |2        |null       |null      |null    |Customer Cancellation|
|10      |1        |1/11/2020 18:50|10km      |10minutes|null              |



4. ### `pizza_names`

|pizza_id|pizza_name|
|--------|----------|
|1       |Meat Lovers|
|2       |Vegetarian|



5. ### `pizza_recipes`


Each `pizza_id` has a standard set of `toppings` which are used as part of the pizza recipe.


|pizza_id|toppings |
|--------|---------|
|1       |1, 2, 3, 4, 5, 6, 8, 10| 
|2       |4, 6, 7, 9, 11, 12| 

6. ### `pizza_toppings`

This table contains all of the `topping_name` values with their corresponding `topping_id` value.


|topping_id|topping_name|
|----------|------------|
|1         |Bacon       | 
|2         |BBQ Sauce   | 
|3         |Beef        |  
|4         |Cheese      |  
|5         |Chicken     |     
|6         |Mushrooms   |  
|7         |Onions      |     
|8         |Pepperoni   | 
|9         |Peppers     |   
|10        |Salami      | 
|11        |Tomatoes    | 
|12        |Tomato Sauce|

---

##  **Data Preprocessing**

### **Data Issues**

Data issues in the existing schema include:

* `customer_orders` table 
  - `null` values entered as text
  - using both `NaN` and `null` values
* `runner_orders` table
  - `null` values entered as text
  - using both `NaN` and `null` values
  - units manually entered in `distance` and `duration` columns

### **Data Cleaning**

`customer_orders`
- Converting `null` and `NaN` values into blanks `''` in `exclusions` and `extras`
- Blanks indicate that the customer requested no extras/exclusions for the pizza, whereas `null` values would be ambiguous.
- Saving the transformations in a temporary table
- We want to avoid permanently changing the raw data via `UPDATE` commands if possible.

`runner_orders`

- Converting `'null'` text values into null values for `pickup_time`, `distance` and `duration`
- Extracting only numbers and decimal spaces for the distance and duration columns
- Use regular expressions and `NULLIF` to convert non-numeric entries to null values
- Converting blanks, `'null'` and `NaN` into null values for cancellation
- Saving the transformations in a temporary table.

**Result:**

`updated_customer_orders`


|order_id|customer_id|pizza_id|exclusions|extras|order_time              |
|--------|-----------|--------|----------|------|------------------------|
|1       |101        |1       |          |      |2020-01-01T18:05:02.000Z|
|2       |101        |1       |          |      |2020-01-01T19:00:52.000Z|
|3       |102        |1       |          |      |2020-01-02T12:51:23.000Z|
|3       |102        |2       |          |      |2020-01-02T12:51:23.000Z|
|4       |103        |1       |4         |      |2020-01-04T13:23:46.000Z|
|4       |103        |1       |4         |      |2020-01-04T13:23:46.000Z|
|4       |103        |2       |4         |      |2020-01-04T13:23:46.000Z|
|5       |104        |1       |          |1     |2020-01-08T21:00:29.000Z|
|6       |101        |2       |          |      |2020-01-08T21:03:13.000Z|
|7       |105        |2       |          |1     |2020-01-08T21:20:29.000Z|
|8       |102        |1       |          |      |2020-01-09T23:54:33.000Z|
|9       |103        |1       |4         |1, 5  |2020-01-10T11:22:59.000Z|
|10      |104        |1       |          |      |2020-01-11T18:34:49.000Z|
|10      |104        |1       |2, 6      |1, 4  |2020-01-11T18:34:49.000Z|

`updated_runner_orders`

| order_id | runner_id | pickup_time         | distance | duration | cancellation            |
|----------|-----------|---------------------|----------|----------|-------------------------|
| 1        | 1         | 2020-01-01 18:15:34 | 20       | 32       |                         |
| 2        | 1         | 2020-01-01 19:10:54 | 20       | 27       |                         |
| 3        | 1         | 2020-01-02 00:12:37 | 13.4     | 20       |                         |
| 4        | 2         | 2020-01-04 13:53:03 | 23.4     | 40       |                         |
| 5        | 3         | 2020-01-08 21:10:57 | 10       | 15       |                         |
| 6        | 3         |                     |          |          | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25       | 25       |                         |
| 8        | 2         | 2020-01-10 00:15:02 | 23.4     | 15       |                         |
| 9        | 2         |                     |          |          | Customer Cancellation   |
| 10       | 1         | 2020-01-11 18:50:20 | 10       | 10       |                         |

---
## **Questions**
  
### Pizza Metrics Questions.
	
1. How many pizzas were ordered?
2. How many unique customer orders were made?
3. How many successful orders were delivered by each runner?
4. How many of each type of pizza was delivered?
5. How many Vegetarian and Meatlovers were ordered by each customer?
6. What was the maximum number of pizzas delivered in a single order?
7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
8. How many pizzas were delivered that had both exclusions and extras?
9. What was the total volume of pizzas ordered for each hour of the day?
10. What was the volume of orders for each day of the week?
### Runner and Customer Experience Questions.
	
1. How many runners signed up for each 1 week? (i.e. week starts 2021-01-01)
2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pick up the order?
3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
4. What was the average distance travelled for each runner?
5. What was the difference between the longest and shortest delivery times for all orders?
6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
7. What is the successful delivery percentage for each runner?



## **Solutions**

### Pizza Metrics Solution:

### 1. How many pizzas were ordered?
```sql	
SELECT COUNT(*) AS pizza_count
FROM updated_customer_orders;
```


|pizza_count|
|-----------|
|14         |

### 2. How many unique customer orders were made?
```sql
SELECT COUNT (DISTINCT order_id) AS order_count
FROM updated_customer_orders;
```

|order_count|
|-----------|
|10         |

### 3. How many successful orders were delivered by each runner?
```sql
SELECT
  runner_id,
  COUNT(order_id) AS successful_orders
FROM updated_runner_orders
WHERE cancellation IS NULL
OR cancellation NOT IN ('Restaurant Cancellation', 'Customer Cancellation')
GROUP BY runner_id
ORDER BY successful_orders DESC;
```

| runner_id | successful_orders |
|-----------|-------------------|
| 1         | 4                 |
| 2         | 3                 |
| 3         | 1                 |

### 4. How many of each type of pizza was delivered?
```sql
SELECT
  pn.pizza_name,
  COUNT(co.*) AS pizza_type_count
FROM updated_customer_orders AS co
INNER JOIN pizza_runner.pizza_names AS pn
   ON co.pizza_id = pn.pizza_id
INNER JOIN pizza_runner.runner_orders AS ro
   ON co.order_id = ro.order_id
WHERE cancellation IS NULL
OR cancellation NOT IN ('Restaurant Cancellation', 'Customer Cancellation')
GROUP BY pn.pizza_name
ORDER BY pn.pizza_name;
```

| pizza_name | pizza_type_count |
|------------|------------------|
| Meatlovers | 9                |
| Vegetarian | 3                |

### 5. How many Vegetarian and Meatlovers were ordered by each customer?
```sql
SELECT
  customer_id,
  SUM(CASE WHEN pizza_id = 1 THEN 1 ELSE 0 END) AS meat_lovers,
  SUM(CASE WHEN pizza_id = 2 THEN 1 ELSE 0 END) AS vegetarian
FROM updated_customer_orders
GROUP BY customer_id;
```

| customer_id | meat_lovers | vegetarian |
|-------------|-------------|------------|
| 101         | 2           | 1          |
| 103         | 3           | 1          |
| 104         | 3           | 0          |
| 105         | 0           | 1          |
| 102         | 2           | 1          |

### 6. What was the maximum number of pizzas delivered in a single order?
```sql
SELECT MAX(pizza_count) AS max_count
FROM (
  SELECT
    co.order_id,
    COUNT(co.pizza_id) AS pizza_count
  FROM updated_customer_orders AS co
  INNER JOIN updated_runner_orders AS ro
    ON co.order_id = ro.order_id
  WHERE 
    ro.cancellation IS NULL
    OR ro.cancellation NOT IN ('Restaurant Cancellation', 'Customer Cancellation')
  GROUP BY co.order_id) AS mycount;
  ```


| max_count |
|-----------|
| 3         |

### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```sql
SELECT 
  co.customer_id,
  SUM (CASE WHEN co.exclusions IS NOT NULL OR co.extras IS NOT NULL THEN 1 ELSE 0 END) AS changes,
  SUM (CASE WHEN co.exclusions IS NULL OR co.extras IS NULL THEN 1 ELSE 0 END) AS no_change
FROM updated_customer_orders AS co
INNER JOIN updated_runner_orders AS ro
  ON co.order_id = ro.order_id
WHERE ro.cancellation IS NULL
  OR ro.cancellation NOT IN ('Restaurant Cancellation', 'Customer Cancellation')
GROUP BY co.customer_id
ORDER BY co.customer_id;
```


| customer_id | changes | no_change |
|-------------|---------|-----------|
| 101         | 0       | 2         |
| 102         | 0       | 3         |
| 103         | 3       | 3         |
| 104         | 2       | 2         |
| 105         | 1       | 1         |

### 8. How many pizzas were delivered that had both exclusions and extras?

```sql
SELECT
  SUM(CASE WHEN co.exclusions IS NOT NULL AND co.extras IS NOT NULL THEN 1 ELSE 0 END) as pizza_count
FROM updated_customer_orders AS co
INNER JOIN updated_runner_orders AS ro
  ON co.order_id = ro.order_id
WHERE ro.cancellation IS NULL
  OR ro.cancellation NOT IN ('Restaurant Cancellation', 'Customer Cancellation')
  ```

| pizza_count |
|-------------|
| 1           |

### 9. What was the total volume of pizzas ordered for each hour of the day?
```sql
SELECT
  DATE_PART('hour', order_time::TIMESTAMP) AS hour_of_day,
  COUNT(*) AS pizza_count
FROM updated_customer_orders
WHERE order_time IS NOT NULL
GROUP BY hour_of_day
ORDER BY hour_of_day;
```


| hour_of_day | pizza_count |
|-------------|-------------|
| 11          | 1           |
| 12          | 2           |
| 13          | 3           |
| 18          | 3           |
| 19          | 1           |
| 21          | 3           |
| 23          | 1           |

### 10. What was the volume of orders for each day of the week?
```sql
SELECT
  TO_CHAR(order_time, 'Day') AS day_of_week,
  COUNT(*) AS pizza_count
FROM updated_customer_orders
GROUP BY 
  day_of_week, 
  DATE_PART('dow', order_time)
ORDER BY day_of_week;
```


| day_of_week | pizza_count |
|-------------|-------------|
| Friday      | 1           |
| Saturday    | 5           |
| Thursday    | 3           |
| Wednesday   | 5           |

### Runner and Customer Experience Solution :

### 1. How many runners signed up for each 1 week? 
```sql
WITH runner_signups AS (
  SELECT
    runner_id,
    registration_date,
    registration_date - ((registration_date - '2021-01-01') % 7)  AS start_of_week
  FROM pizza_runner.runners
)
SELECT
  start_of_week,
  COUNT(runner_id) AS signups
FROM runner_signups
GROUP BY start_of_week
ORDER BY start_of_week;
```


| start_of_week            | signups |
|--------------------------|---------|
| 2021-01-01T00:00:00.000Z | 2       |
| 2021-01-08T00:00:00.000Z | 1       |
| 2021-01-15T00:00:00.000Z | 1       |

### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
```sql
WITH runner_pickups AS (
  SELECT
    ro.runner_id,
    ro.order_id,
    co.order_time,
    ro.pickup_time,
    (pickup_time - order_time) AS time_to_pickup
  FROM updated_runner_orders AS ro
  INNER JOIN updated_customer_orders AS co
    ON ro.order_id = co.order_id
)
SELECT 
  runner_id,
  date_part('minutes', AVG(time_to_pickup)) AS avg_arrival_minutes
FROM runner_pickups
GROUP BY runner_id
ORDER BY runner_id;
```

| runner_id | avg_arrival_minutes |
|-----------|---------------------|
| 1         | -4                  |
| 2         | 23                  |
| 3         | 10                  |

### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
```sql
WITH order_count AS (
  SELECT
    order_id,
    order_time,
    COUNT(pizza_id) AS pizzas_order_count
  FROM updated_customer_orders
  GROUP BY order_id, order_time
), 
prepare_time AS (
  SELECT
    ro.order_id,
    co.order_time,
    ro.pickup_time,
    co.pizzas_order_count,
    (pickup_time - order_time) AS time_to_pickup
  FROM updated_runner_orders AS ro
  INNER JOIN order_count AS co
    ON ro.order_id = co.order_id
  WHERE pickup_time IS NOT NULL
)

SELECT
  pizzas_order_count,
  AVG(time_to_pickup) AS avg_time
FROM prepare_time
GROUP BY pizzas_order_count
ORDER BY pizzas_order_count;
```

| pizzas_order_count | avg_time        |
|--------------------|-----------------|
| 1                  | 12              |
| 2                  | -6              |
| 3                  | 29              |

### 4. What was the average distance travelled for each runner?
```sql
SELECT
  runner_id,
  ROUND(AVG(distance), 2) AS avg_distance
FROM updated_runner_orders
GROUP BY runner_id
ORDER BY runner_id;
```

| runner_id | avg_distance |
|-----------|--------------|
| 1         | 15.85        |
| 2         | 23.93        |
| 3         | 10.00        |

### 5. What was the difference between the longest and shortest delivery times for all orders?
```sql
SELECT
  MAX(duration) - MIN(duration) AS difference
FROM updated_runner_orders;
```

| difference |
|------------|
| 30         |

### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
```sql
WITH order_count AS (
  SELECT
    order_id,
    order_time,
    COUNT(pizza_id) AS pizzas_count
  FROM updated_customer_orders
  GROUP BY 
    order_id, 
    order_time
)
  SELECT
    ro.order_id,
    ro.runner_id,
    co.pizzas_count,
    ro.distance,
    ro.duration,
    ROUND(60 * ro.distance / ro.duration, 2) AS speed
  FROM updated_runner_orders AS ro
  INNER JOIN order_count AS co
    ON ro.order_id = co.order_id
  WHERE pickup_time IS NOT NULL
  ORDER BY speed DESC;
  ```

| order_id | runner_id | pizzas_count | distance | duration | speed |
|----------|-----------|--------------|----------|----------|-------|
| 8        | 2         | 1            | 23.4     | 15       | 93.60 |
| 7        | 2         | 1            | 25       | 25       | 60.00 |
| 10       | 1         | 2            | 10       | 10       | 60.00 |
| 2        | 1         | 1            | 20       | 27       | 44.44 |
| 3        | 1         | 2            | 13.4     | 20       | 40.20 |
| 5        | 3         | 1            | 10       | 15       | 40.00 |
| 1        | 1         | 1            | 20       | 32       | 37.50 |
| 4        | 2         | 3            | 23.4     | 40       | 35.10 |

**Finding:**
- **Orders shown in decreasing order of average speed:**
> *While the fastest order only carried 1 pizza and the slowest order carried 3 pizzas, there is no clear trend that more pizzas slow down the delivery speed of an order.*  

### 7. What is the successful delivery percentage for each runner?
```sql
SELECT
  runner_id,
  COUNT(pickup_time) as delivered,
  COUNT(order_id) AS total,
  ROUND(100 * COUNT(pickup_time) / COUNT(order_id)) AS delivery_percent
FROM updated_runner_orders
GROUP BY runner_id
ORDER BY runner_id;
```


| runner_id | delivered | total | delivery_percent |
|-----------|-----------|-------|------------------|
| 1         | 4         | 4     | 100              |
| 2         | 3         | 4     | 75               |
| 3         | 1         | 2     | 50               |


## **Recommedations**

### **1. Optimise Runner Efficiency**
**Analyse Pickup:** The average time it takes for runners to arrive at the headquarters varies widely, with some negative values suggesting data issues or inaccuracies. It's crucial to correct these data anomalies and focus on training or scheduling improvements to ensure more consistent and quicker pickups.

**Speed Analysis:** The average speed analysis indicates that there's no clear trend between the number of pizzas delivered and the delivery speed. However, runners with higher average speeds might be more efficient. Consider incentivizing high-performing runners to maintain or improve their performance.

### **2. Improve Customer Experience**
**Order Preparation Time:** The relationship between the number of pizzas and preparation time is inconsistent. Investigate the reasons behind the negative average time for some orders, as this might point to data issues or inefficiencies in the kitchen. Ensuring accurate time tracking and addressing any bottlenecks in pizza preparation can enhance overall customer satisfaction.

**Delivery Time Consistency:** The difference between the longest and shortest delivery times is significant. Implement strategies to minimize delivery times, such as optimising delivery routes or increasing the number of runners during peak hours.

### **3. Enhance Marketing and Promotions**
**Order Volume by Hour and Day:** Pizza orders peak at certain hours and days. Utilise this information to tailor marketing campaigns and promotions. For instance, offering special deals during slower periods (e.g., Fridays and weekends) might help balance out order volume and increase revenue.

**Customer Segmentation:** Analysing how many of each type of pizza customers order and their preferences for exclusions and extras can help in creating personalised promotions and targeted marketing. This segmentation can lead to better customer engagement and higher sales.

### **4. Optimise Delivery Operations**
**Distance and Speed Tracking:** The average distance traveled varies by runner. Ensure that delivery routes are optimized for efficiency to reduce average travel distance and improve delivery speed. Implementing or refining a route optimization system could be beneficial.

**Successful Deliveries:** The successful delivery percentage shows that while most runners perform well, there's room for improvement. Investigate the reasons behind cancellations and focus on minimising these occurrences. Providing additional training or support to runners with lower success rates may improve overall performance.

### **5. Expand Runner Base**

**Recruitment Trends:** Analyse runner sign-ups by week and strategize recruitment efforts based on observed patterns. For example, if sign-ups typically increase after certain dates or events, plan recruitment drives around these times to ensure adequate coverage and avoid potential delivery issues.

### **6. Leverage Customer Feedback**
**Feedback Collection:** Collect and analyse customer feedback to identify areas of improvement in both the pizza quality and delivery service. Implementing a feedback loop will help in making data-driven decisions to enhance overall customer satisfaction.
