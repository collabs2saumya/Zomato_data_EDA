Zomato Data Analysis Project using SQL
Project Overview
This project presents an in-depth analysis of a Zomato-like food delivery platform dataset using PostgreSQL. The objective is to design a comprehensive relational database schema, clean and preprocess data for inconsistencies, and execute complex SQL queries to solve critical business problems. The analysis provides actionable insights into customer behavior, restaurant performance, delivery rider efficiency, and revenue optimization trends.

Database Schema & Design
The architecture consists of 5 core tables carefully structured with primary keys, foreign keys, and analytical constraints to ensure data integrity.

Entity Relationship Summary
customers: Stores registered customer profiles.

restaurents: Contains details of various restaurant partners across different cities.

orders: Records core transactional details of orders placed by customers.

riders: Tracks delivery rider sign-ups and metadata.

deliveries: Logistical logs containing statuses and time metrics of completed/failed deliveries.

```--Zomato Data Analysis
CREATE TABLE customers
(
customer_id INT PRIMARY KEY,
customer_name VARCHAR(25),
reg_date DATE
);


CREATE TABLE restaurents
(
restaurant_id INT PRIMARY KEY	,
restaurant_name	 VARCHAR(55) ,
city VARCHAR(15)	 ,
opening_hours VARCHAR(55) 
);

CREATE TABLE orders
(
order_id INT PRIMARY KEY,
customer_id INT	,
restaurant_id INT	,
order_item VARCHAR(55),
order_date DATE	,
order_time TIME	,
order_status VARCHAR(55),
total_amount FLOAT
);

--Adding Fk Constraint
ALTER TABLE orders
ADD CONSTRAINT fk_customers
FOREIGN KEY (customer_id)
REFERENCES customers(customer_id)

ALTER TABLE orders
ADD CONSTRAINT fk_restaurents
FOREIGN KEY (restaurant_id)
REFERENCES restaurents(restaurant_id);

CREATE TABLE riders
(
rider_id INT PRIMARY KEY	,
rider_name VARCHAR(55)	,
sign_up DATE
);


-- DROP TABLE  IF EXISTS deliveries;
CREATE TABLE deliveries
(
delivery_id INT PRIMARY KEY 	,
order_id INT	,
delivery_status VARCHAR(35)	,
delivery_time TIME	,
rider_id INT,
CONSTRAINT fk_orders FOREIGN KEY (order_id) 
REFERENCES orders(order_id),
CONSTRAINT fk_riders FOREIGN KEY (rider_id) 
REFERENCES riders(rider_id)
);

--------------------------------------------------------------
--TRUNCATE TABLE orders CASCADE;
--EDA
--Heirarchical data Insertion or Import Datasets
select * from customers;
select * from restaurents;
select * from orders;
select * from riders;
select * from deliveries;
---------------------------------------------------------------
--CHECKING & HANDLING NULL VALUES
select count(*) from customers
where customer_id is NULL
  or
      customer_name is NULL
  or
      reg_date is NULL

SELECT count(*)
FROM restaurents
WHERE ROW(restaurant_id, restaurant_name, city, opening_hours) 
      IS NULL;

SELECT count(*)
FROM orders
WHERE ROW(orders.*) IS NULL;	  
--THIS FORMAT RETURN COUNT WHEN EACH COLUMN HAS NULL VALUES OTHERWISE NOT RETURN 

SELECT count(*)
FROM deliveries
WHERE ROW(deliveries.*) IS NULL;

SELECT count(*)
FROM riders
WHERE ROW(riders.*) IS NULL;


INSERT INTO orders(order_id,customer_id,restaurant_id)
VALUES 
(193,6,54),
(151,8,41),
(364,7,35);

-- SELECT COUNT(*)
-- FROM orders
-- WHERE EXISTS (
--     SELECT *
--     FROM jsonb_each(to_jsonb(orders))
--     WHERE value IS NULL
-- );

SELECT COUNT(*)
FROM orders
WHERE EXISTS (
    SELECT 1
    FROM jsonb_each(to_jsonb(orders))
    WHERE value = 'null'::jsonb
);

DELETE FROM orders
WHERE EXISTS (
    SELECT 1
    FROM jsonb_each(to_jsonb(orders))
    WHERE value = 'null'::jsonb
);
----------------------------------------------------------------------------
--ANALYSIS & REPORTS

--Q1- WRITE A QUERY TO FIND TOP FREQUENTLY ORDERED DISHES BY CUSTOMERS CALLED
-- "Nisha Chaudhary" IN LAST 3 YEAR

--join cx and orders , 
--filter for last 1 year , 
-- filter 'Arjun Mehta'
--group by cx id , dishes , cnt ,

select 
       customer_name,
	   dishes,
	   total_orders,rank
from
(select 
      c.customer_id,
      c.customer_name,
      o.order_item as dishes,
      COUNT(*) as total_orders,
	  DENSE_RANK() OVER(ORDER BY COUNT(*) DESC) as rank
from orders as o
Join customers as c
ON c.customer_id = o.customer_id
where 
      o.order_date >= CURRENT_DATE - INTERVAL '3 Year'
	  and 
	  c.customer_name = 'Nisha Chaudhary'
Group by 1,2,3
order by 1,4 DESC
) as t1


-- POPULAR TIME SLOTS
--Q2 IDENTIFY THE TIME SLOTS DURING WHICH THE MOST ORDERS ARE PLACED , BASED ON 
-- 2 HOUR INTERVALS

 select 
       CASE 
    WHEN EXTRACT(HOUR FROM order_time) BETWEEN 0 AND 1 THEN '00:00 - 02:00'

    WHEN EXTRACT(HOUR FROM order_time) BETWEEN 2 AND 3 THEN '02:00 - 04:00' 

    WHEN EXTRACT(HOUR FROM order_time) BETWEEN 4 AND 5  THEN '04:00 - 06:00'

    WHEN EXTRACT(HOUR FROM order_time) BETWEEN 6 AND 7 THEN '06:00 - 08:00'

    WHEN EXTRACT(HOUR FROM order_time) BETWEEN 8 AND 9  THEN '08:00 - 10:00'

    WHEN EXTRACT(HOUR FROM order_time) BETWEEN 10 AND 11 THEN '10:00 - 12:00'

    WHEN EXTRACT(HOUR FROM order_time) BETWEEN 12 AND 13 THEN '12:00 - 14:00'

    WHEN EXTRACT(HOUR FROM order_time) BETWEEN 14 AND 15 THEN '14:00 - 16:00'

    WHEN EXTRACT(HOUR FROM order_time) BETWEEN 16 AND 17 THEN '16:00 - 18:00'

    WHEN EXTRACT(HOUR FROM order_time) BETWEEN 18 AND 19 THEN '18:00 - 20:00'

    WHEN EXTRACT(HOUR FROM order_time) BETWEEN 20 AND 21 THEN '20:00 - 22:00'

    WHEN EXTRACT(HOUR FROM order_time) BETWEEN 22 AND 23 THEN '22:00 - 24:00'
END AS time_slot,
COUNT(order_id) as order_count
From orders
Group by time_slot
order by order_count desc;

------------Approach 2 

select 
       FLOOR(EXTRACT (HOUR FROM order_time)/2 )*2 as start_time,
       FLOOR(EXTRACT (HOUR FROM order_time)/2 )*2 +2 as end_time,
	   COUNT(*) AS total_orders
from orders
GROUP BY 1 ,2
ORDER BY 3 DESC;


--ORDER VALUE ANALYSIS
--Q3 FIND THE AVERAGE ORDER VALUE PER CUSTOMER WHO HAS PLACED MORE THAN 4
--ORDERS , RETURN customer_name and aov(average order value)

select 
        c.customer_name,
		o.customer_id,
       avg(total_amount) as aov,
	   count(order_id) as total_orders,
	   DENSE_RANK() OVER(order by count(*) desc) as rank
from orders as o
join customers as c
on c.customer_id = o.customer_id
group by 1,2
having count(order_id) > 4

--HIGH VALUE CUSTOMERS
-- Q4 LIST THE CUSTOMERS WHO HAVE SPENT MORE THAN 5000rs IN TOTAL ON FOOD ORDERS
-- RETURN CUSTOMER_NAME AND CUSTOMER_IDL;

select 
        c.customer_name,
	   sum(o.total_amount) as total_spent
from orders as o
join customers as c
on c.customer_id = o.customer_id
group by 1
 having  sum(o.total_amount) > 10000
 order by 2 desc


--ORDERS WITHOUT DELIVERY
--Q5 WRITE QUERY TO FIND ORDERS THAT WERE PLACED BUT NOT DELIVERED
--RETURN EACH RESTAURANT NAME , CITY , NUMBER OF NOT DELIEVERED ORDERS

 
select r.restaurant_name,
       count(o.order_id) as cnt_not_delivered
from orders as o
left join 
restaurents as r
on r.restaurant_id = o.restaurant_id
left join deliveries as d
on d.order_id = o.order_id
where d.delivery_id is NULL
group by 1

--Q6 Rank restaurants by revenue within each city

WITH ranking_table as
(
SELECT
    r.city,
    r.restaurant_name,
    SUM(o.total_amount) AS revenue,
    RANK() OVER (PARTITION BY r.city ORDER BY SUM(o.total_amount) DESC) AS rank
FROM orders AS o
JOIN restaurents as r
  ON r.restaurant_id = o.restaurant_id
GROUP BY 1,2
)
SELECT * 
FROM ranking_table
where rank = 1


--Q7 Most popular dish per city based on number of orders

select * 
from
(SELECT
    r.city,
	o.order_item as dish,
	COUNT(order_id) as total_orders,
    RANK() OVER (PARTITION BY r.city ORDER BY COUNT(order_id) DESC) AS rank
FROM orders AS o
JOIN restaurents as r
  ON r.restaurant_id = o.restaurant_id
GROUP BY 1,2
) as t1
where rank = 1

--Q8 FIND CUSTOMERS WHO HAVEN"T PLACED AN ORDER IN 2024 BUT DID IN 2023
SELECT DISTINCT customer_id FROM orders
WHERE EXTRACT(YEAR FROM order_date) = 2023 
      and 
	  customer_id NOT IN 
	  (SELECT DISTINCT customer_id FROM orders
        WHERE EXTRACT(YEAR FROM order_date) = 2024)


--CANCELLATION RATE COMPARISON
--Q9 CALCULATE AND COMPARE THE ORDER CANCELLATION RATE FOR EACH RESTAURANT BETWEEN
    -- CURRENT YEAR AND PREVIOUS YEAR
WITH cancel_rate
AS
(
SELECT 
      restaurant_id,
	  COUNT(o.order_id) as total_orders,
	  COUNT(CASE WHEN d.delivery_id IS NULL THEN 1 END) not_delivered
FROM orders as o
LEFT JOIN
deliveries as d
on o.order_id = d.order_id
WHERE EXTRACT(YEAR FROM order_date) = 2023
GROUP BY 1
),
last_year_data
AS
(
SELECT 
      restaurant_id,
	  total_orders,
      not_delivered,
	  Round(not_delivered::numeric/total_orders::numeric *100,2) as cancellation_ratio
FROM cancel_rate
)
SELECT * FROM last_year_data;

--
WITH cancel_rate
AS
(
SELECT 
      restaurant_id,
	  COUNT(o.order_id) as total_orders,
	  COUNT(CASE WHEN d.delivery_id IS NULL THEN 1 END) not_delivered
FROM orders as o
LEFT JOIN
deliveries as d
on o.order_id = d.order_id
WHERE EXTRACT(YEAR FROM order_date) = 2024
GROUP BY 1
),
last_year_data
AS
(
SELECT 
      restaurant_id,
	  total_orders,
      not_delivered,
	  Round(not_delivered::numeric/total_orders::numeric *100,2) as cancellation_ratio
FROM cancel_rate
)
SELECT * FROM last_year_data;


--Q10 RIDER AVERAGE DELIVERY TIME
-- DETERMINE EACH RIDERS AVERAGE DELIVERY TIME

SELECT 
       o.order_id,
	    o.order_time,
		d.delivery_time,
       rider_id,
	   o.order_time - d.delivery_time AS time_diff,
	   EXTRACT(EPOCH FROM (d.delivery_time - o.order_time +
	   CASE WHEN d.delivery_time < o.order_time THEN INTERVAL '1 DAY' ELSE
	   INTERVAL '0 DAY' END))/ 60 AS time_diff_insec
FROM orders as o 
JOIN deliveries as d 
ON o.order_id = d.order_id
WHERE d.delivery_status = 'Delivered'


--Q11 MONTHLY RESTAURANT GROWTH RATIO
--CALCULATE EACH RESTAURANT GROWTH RATIO BASED ON THE TOTAL NUMBER OF DELIVERED 
--ORDERS SINCE ITS JOINING


WITH GROWTH_RATIO
AS
(
SELECT 
       o.restaurant_id,
	   TO_CHAR(o.order_date,'mm-yy') as month,
	   COUNT(o.order_id) as cr_month_orders,
	   LAG (COUNT(o.order_id),1) OVER(PARTITION BY  o.restaurant_id 
	   ORDER BY TO_CHAR(o.order_date,'mm-yy')) as prev_month_orders
FROM ORDERS as o
JOIn 
deliveries as d 
ON o.order_id = d.order_id
WHERE d.delivery_status = 'Delivered'
GROUP BY 1,2
order by 1,2
)
SELECT 
       restaurant_id,
	   month,
	   prev_month_orders,
	   cr_month_orders,
	ROUND((cr_month_orders::NUMERIC - prev_month_orders::NUMERIC) /prev_month_orders::numeric +100,2)
	as growth_ratio
FROM GROWTH_RATIO	   


--Q12 CUSTOMER SEGMENTATION
--CUSTOMER SEGMENTATION : SEGMENT CUSTOMERS INTO GOLD OR SILVER GROUPS BASED 
--ON THEIR TOTAL SPENDINGS , COMPARED TO AOV , IF CUSTOMER TOTAL SPENDING EXCEEDS AOV
-- LABEL THEM AS GOLD OTHERVISE SILVER 
--QUERY TO DETERMINE EACH SEGMENTS TOTAL NUMBER OF ORDERS AND TOTAL REVENUE

SELECT 
       cx_category,
	   SUM(totaL_orders) as total_orders,
	   SUM(total_spent) as total_revenue
FROM(	   
SELECT 
      customer_id,
	  SUM(total_amount) as total_spent,
	  COUNT(order_id) as total_orders,
	  CASE 
	      WHEN SUM(total_amount) > (SELECT AVG(total_amount) FROM orders) THEN 'GOLD'
		  ELSE 'SILVER'
	   END AS cx_category
	   FROM orders
	   GROUP BY 1
) AS T1
GROUP BY 1


--Q13 RIDER MONTHLY EARNING :
--CALCULATE EACH RIDERS TOTAL MONTHLY EARNINGS,ASSUMING THEY EARN 8% OF ORDER AMOUNT

SELECT 
      d.rider_id,
	  TO_CHAR(o.order_date ,'mm-yy') as month,
	  SUM(total_amount) as revenue,
	  SUM(total_amount) * 0.08 as riders_earnings
FROM orders as o 
JOIN deliveries as d 
ON o.order_id = d.order_id
GROUP BY 1,2
ORDER BY 1,2


--Q14 RIDER RATING ANALYSIS
--FIND NUMBER OF 5 STAR, 4 STAR AND 3 STAR EACH RIDER HAS BASED ON DELIVERY TIME
-- DELIVERED LESS THAN 15 MIN - 5 STAR , 15-20 MIN - 4STAR , ABOVE 20MIN - 3 STAR

SELECT 
      rider_id,
	  stars,
	  COUNT(*) AS total_stars
FROM	  
(SELECT 
      rider_id,
	  delivered_time,
	  CASE 
	      WHEN delivered_time  < 15 THEN '5 star'
		  WHEN  delivered_time BETWEEN 15 AND 20 THEN '4 star'
		  ELSE '3 star'
	  END AS STARS	  
FROM
(
SELECT 
       o.order_id,
	   o.order_time,
	   d.delivery_time,
	   EXTRACT(EPOCH FROM ( d.delivery_time - o.order_time +
	   CASE WHEN d.delivery_time < o.order_time THEN INTERVAL '1 DAY'
	   ELSE INTERVAL '0 DAY' END)) /60 AS delivered_time,
	   d.rider_id
FROM orders as o
JOIN deliveries as d 
ON o.order_id = d.order_id
WHERE delivery_status = 'Delivered'
) AS t1
) AS t2
GROUP BY 1 ,2
ORDER BY 1 ,3 DESC



--Q15 ORDER FREQUENCY BY DAY
-- ANALYSE ORDER FREQUENCY PER DAY OF WEEK AND IDENTIFY THE PEAK DAY FOR EACH RESTAURANT

SELECT * 
FROM
(
SELECT 
       restaurant_name,
	   -- o.order_date,
	   TO_CHAR(o.order_date,'DAY') as day,
	   COUNT(o.order_id) AS total_orders ,
	   RANK() OVER(PARTITION BY restaurant_name ORDER BY COUNT(o.order_id)DESC) as rank
FROM orders as o
JOIN 
restaurents as r
ON o.restaurant_id = r.restaurant_id
GROUP BY 1,2
ORDER BY 1 ,3 DESC
) AS t1
WHERE RANK = 1


--Q16 CUSTOMER LIFETIME VALUE
-- CALCULATE THE TOTAL REVENUE GENERATED BY EACH CUSTOMER OVER ALL THEIR ORDERS

SELECT 
      o.customer_id,
	  c.customer_name,
	  SUM(o.total_amount) as CLV
FROM ORDERS AS o
JOIN customers as c
ON o.customer_id = c.customer_id
GROUP BY 1,2


-- Q17 MONTHLY SALES TRENDS
--IDENTIFY SALES TRENDS BY COMPARING EACH MONTHS TOTAL SALES TO PREVIOUS MONTH

SELECT 
       EXTRACT(YEAR FROM order_date) as year,
	   EXTRACT(MONTH FROM order_date) as month,
       SUM(total_amount) as total_sale,
	   LAG(SUM(total_amount),1) OVER(ORDER BY EXTRACT(YEAR FROM order_date),
	   EXTRACT(MONTH FROM order_date)) AS prev_month_sale
	   
 FROM orders
 GROUP BY 1,2
 ORDER BY 1,2


--Q18 RIDER EFFICIENCY 
-- EVALUATE RIDER EFFICIENCY BY DETERMINING AVERAGE DELIVERY TIMES AND IDENTIFYING
--THOSE WITH LOWEST AND HIGHEST

WITH new_table
as
(
SELECT *,
      d.rider_id as riders_id,
	  EXTRACT(EPOCH FROM (d.delivery_time - o.order_time +
	   CASE WHEN d.delivery_time < o.order_time THEN INTERVAL '1 DAY' ELSE
	   INTERVAL '0 DAY' END))/ 60 AS time_deliver
FROM ORDERS as o
JOIN deliveries as d 
ON o.order_id = d.order_id
WHERE d.delivery_status = 'Delivered'
),

riders_time
as 
(
SELECT 
      riders_id,
	  AVG(time_deliver) as avg_time
from 
new_table
GROUP BY 1
)

SELECT  
      MIN(avg_time),
	  MAX(avg_time)
FROM riders_time	  
      



--Q19 ORDER ITEM POPULARITY
--TRACK THE POPULARITY OF SPECIFIC ORDER ITEMS OVER TIME AND IDENTIFY SEASONAL 
--DEMAND SPICES

SELECT 
       order_item,
	   seasons,
	   COUNT(order_id) as total_orders
from	   
(
SELECT 
      *,
	  EXTRACT(MONTH FROM order_date) as month,
	  CASE
	       WHEN EXTRACT(MONTH FROM order_date) BETWEEN 4 AND 6 THEN 'SPRING'
		   WHEN EXTRACT(MONTH FROM order_date) >6 AND 
		   EXTRACT(MONTH FROM order_date) < 9 THEN 'SUMMER'
		   ELSE 'WINTER '
	   END AS seasons	   
FROM orders
) as t1
GROUP BY 1,2
ORDER BY 1,3 DESC


--Q20 RANK EACH CITY BASED ON TOTAL REVENUE

SELECT 
      r.city,
	  SUM(total_amount) as total_revenue,
	  RANK() OVER(ORDER BY SUM(total_amount) DESC) as city_rank
FROM orders as o
JOIN restaurents as r
ON o.restaurant_id = r.restaurant_id
GROUP BY 1
..................................................................................................................................................................

```
# 📚 Learning Outcomes

By completing this project, you will learn:

- Advanced SQL
- PostgreSQL
- Window Functions
- Business Analytics
- Data Cleaning
- Customer Segmentation
- Revenue Analysis
- Real-world SQL Problem Solving

---

# 👨‍💻 Author

## Saumya Gupta

B.Tech — Blockchain & Cyber Security  
SQL & Data Analytics Enthusiast

---

# ⭐ If You Like This Project

⭐ Star this repository  
🍴 Fork this repository  
📢 Share with others

---

# 📜 License

This project is open-source and free to use.
