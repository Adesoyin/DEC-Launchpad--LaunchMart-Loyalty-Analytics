# DEC-Launchpad--LaunchMart-Loyalty-Analytics
This project analyzes customer behavior, revenue performance, and loyalty engagement of LaunchMart e-commerce company that recently launched a loyalty program to increase customer retention. Customers earn points when they place orders and extra bonus points during promotions. 

## Project Purpose
1. To create a database tables and insert the sample datasets into each table.
2. To explore the company's customer, orders, and loyalty program data to help the marketing and operations teams make informed decisions.

**Creating Tables**
The following database tables were created using DDL command in Postgres PGAdmin4

- [*] customers,
- [*] products,
- [*] orders,
- [*] order items, and
- [*] loyalty points


      CREATE TABLE customers (

      customer_id SERIAL PRIMARY KEY,
      full_name VARCHAR(50) NOT NULL,
      email VARCHAR(50) UNIQUE,
      join_date DATE NOT NULL
      );
    
      CREATE TABLE products (
      product_id SERIAL PRIMARY KEY,
      product_name VARCHAR(50) NOT NULL,
      category VARCHAR(50) NOT NULL,
      price INTEGER NOT NULL
      );
      
      
      CREATE TABLE orders (
      order_id SERIAL PRIMARY KEY,
      customer_id INTEGER NOT NULL REFERENCES customers(customer_id),
      order_date DATE NOT NULL,
      total_amount INTEGER NOT NULL
      );
      
      
      CREATE TABLE order_items (
      order_item_id SERIAL PRIMARY KEY,
      order_id INTEGER NOT NULL REFERENCES orders(order_id),
      product_id INTEGER NOT NULL REFERENCES products(product_id),
      quantity INTEGER NOT NULL,
      line_total INTEGER NOT NULL
      );
      
      
      CREATE TABLE loyalty_points (
      loyalty_id SERIAL PRIMARY KEY,
      customer_id INTEGER NOT NULL REFERENCES customers(customer_id),
      points_earned INTEGER NOT NULL,
      transaction_date DATE NOT NULL,
      source VARCHAR(50) NOT NULL
      );

**Data Import**

Data were added to each database table using insert command.

**Business Questions & Solutions**

The below questions are being raised and answered by writing Query to understand and make deceisions.

__**1. Count the total number of customers who joined in 2023.**__

    SELECT COUNT(DISTINCT customer_id) CustomersThatJoinedIn2023
    FROM customers
    WHERE EXTRACT(YEAR FROM customers.join_date) = 2023


![alt text](Images/Customers%20that%20joined%20in%202023.png)

__**2. For each customer return customer_id, full_name, total_revenue (sum of total_amount from orders). Sort descending.**__

    --For each customer return customer_id, full_name, 
    --total_revenue (sum of total_amount from orders). Sort descending.

    SELECT c.customer_id, c.full_name, SUM(o.total_amount) total_revenue
    FROM customers AS c
    LEFT JOIN orders AS o
    ON c.customer_id = o.customer_id
    GROUP BY c.customer_id, c.full_name
    ORDER BY total_revenue desc


![alt text](Images/Total%20revenue%20by%20customers.png)


__**2. Return the top 5 customers by total_revenue with their rank.**__

    --Return the top 5 customers by total_revenue with their rank.

    WITH CTE AS (
          SELECT c.customer_id, c.full_name, SUM(o.total_amount) total_revenue, 
          RANK() OVER (order by SUM(o.total_amount) DESC) AS Rank
          FROM customers AS c
          LEFT JOIN orders AS o
          ON c.customer_id = o.customer_id
          GROUP BY c.customer_id, c.full_name
          )
    SELECT * 
    FROM CTE
    LIMIT 5

![alt text](Images/Customer%20Rank.png)