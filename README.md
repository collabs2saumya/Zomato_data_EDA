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

Data Definition Language (DDL) Script
'''-- 1. Create Customers Table
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(25),
    reg_date DATE
);

-- 2. Create Restaurants Table (Note: Spelled as 'restaurents' to match database)
CREATE TABLE restaurents (
    restaurant_id INT PRIMARY KEY,
    restaurant_name VARCHAR(55),
    city VARCHAR(15),
    opening_hours VARCHAR(55)
);

-- 3. Create Orders Table
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    restaurant_id INT,
    order_item VARCHAR(55),
    order_date DATE,
    order_time TIME,
    order_status VARCHAR(55),
    total_amount FLOAT
);

-- Adding Foreign Key Constraints to Orders Table
ALTER TABLE orders
ADD CONSTRAINT fk_customers
FOREIGN KEY (customer_id)
REFERENCES customers(customer_id);

ALTER TABLE orders
ADD CONSTRAINT fk_restaurents
FOREIGN KEY (restaurant_id)
REFERENCES restaurents(restaurant_id);

-- 4. Create Riders Table
CREATE TABLE riders (
    rider_id INT PRIMARY KEY,
    rider_name VARCHAR(55),
    sign_up DATE
);

-- 5. Create Deliveries Table
CREATE TABLE deliveries (
    delivery_id INT PRIMARY KEY,
    order_id INT,
    delivery_status VARCHAR(35),
    delivery_time TIME,
    rider_id INT,
    CONSTRAINT fk_orders FOREIGN KEY (order_id) REFERENCES orders(order_id),
    CONSTRAINT fk_riders FOREIGN KEY (rider_id) REFERENCES riders(rider_id)
);'''
