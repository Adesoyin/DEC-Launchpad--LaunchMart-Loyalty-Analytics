# DEC-Launchpad--LaunchMart-Loyalty-Analytics
This project analyzes customer behavior, revenue performance, and loyalty engagement of LaunchMart e-commerce company that recently launched a loyalty program to increase customer retention. Customers earn points when they place orders and extra bonus points during promotions. 

**Project Purpose**
1. To craete a datbase tables and insert the sample datasets into each table.
2. To explore the company's customer, orders, and loyalty program data to help the marketing and operations teams make informed decisions.

**Tables**
1.  customers,
2.  products,
3.  orders,
4.  order items, and
5.  loyalty points

**Creating Tables**
The DDL command was used in creating the database tables in postgres. 

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

