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

## Business Questions & Solutions

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

__**3. Return the top 5 customers by total_revenue with their rank.**__

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

__**4. Produce a table with year, month, monthly_revenue for all months in 2023 ordered chronologically.**__

    --Produce a table with year, month, monthly_revenue for all months in 2023 ordered chronologically.
    WITH CTE AS (
          SELECT EXTRACT(YEAR FROM order_date) AS Order_Year, 
              TO_CHAR(order_date, 'Month') AS Order_Month,
              EXTRACT(Month FROM order_date) AS month_number,
              SUM(total_amount) AS Monthly_revenue
          FROM orders
          WHERE EXTRACT(YEAR FROM order_date) = 2023
          GROUP BY Order_Year, Order_Month, month_number
          )
    SELECT order_year, order_month, monthly_revenue
    FROM CTE
    ORDER BY order_year,month_number

![alt text](Images/Monthly%20Revenue.png)


### 5. Find customers with no orders in the last 60 days relative to 2023-12-31. Return customer_id, full_name, last_order_date.

It is consider that the last active date in the dataset is 2023-12-31. Hence, the scriot would be detailing the customers that has no order in the last 60 days.

    --Find customers with no orders in the last 60 days relative to 2023-12-31. 
    --Return customer_id, full_name, last_order_date

    WITH customerlastorderdate AS (
                SELECT a.customer_id, a.full_name, MAX(b.order_date) last_order_date
                FROM CUSTOMERS a
                LEFT JOIN orders b
                ON a.customer_id = b.customer_id
                GROUP BY a.customer_id, a.full_name
    ),
    customersin60daysorder AS (
                SELECT * 
                FROM orders
                WHERE order_date >= '2023-12-31'::date - INTERVAL '60 days'
    )

    SELECT * 
    FROM customerlastorderdate AS a
    WHERE NOT EXISTS (
                SELECT order_id 
                FROM customersin60daysorder 
                WHERE customer_id = a.customer_id)
    ORDER BY customer_id

![alt text](Images/Customers%20with%20no%20order%20in%20last%2060%20days.png)

### 6. Calculate average order value (AOV) for each customer: return customer_id, full_name, aov (average total_amount of their orders). Exclude customers with no orders.

    --Calculate average order value (AOV) for each customer: return customer_id, full_name, 
    --aov (average total_amount of their orders). Exclude customers with no orders.
    --Using o.customer_id only return the customers with orders

    WITH CTE AS (
    SELECT a.*,b.product_id, b.product_name, b.category, b.price, c.order_date, 
                        c.customer_id, d.full_name,c.total_amount 
                from order_items a
                left join products b
                ON a.product_id = b.product_id
                left join orders c
                ON a.order_id = c.order_id
                left join customers as d
                on c.customer_id = d.customer_id
    )
        SELECT customer_id, full_name, ROUND(AVG(line_total)) AS average_order_value
        FROM CTE 
        GROUP BY customer_id, full_name
        ORDER BY average_order_value DESC

![alt text](Images/Average%20Order%20by%20Customer.png)

### 7. For all customers who have at least one order, compute customer_id, full_name, total_revenue, spend_rank where spend_rank is a dense rank, highest spender = rank 1.

    --For all customers who have at least one order, compute customer_id, full_name, 
    --total_revenue, spend_rank where spend_rank is a dense rank, highest spender = rank 1.

    WITH CTE AS (
                SELECT a.customer_id, a.full_name, count( distinct b.order_id) ordercount,
                SUM(total_amount) AS total_revenue
                FROM CUSTOMERS a
                LEFT JOIN orders b
                ON a.customer_id = b.customer_id
                GROUP BY a.customer_id, a.full_name
                HAVING count( distinct b.order_id) IS NOT NULL

                
    )
    SELECT customer_id,full_name,total_revenue, 
    CONCAT ('Rank ' , DENSE_RANK() OVER (ORDER BY total_revenue DESC)) spend_rank
    FROM CTE

    or

    WITH CTE AS (
    SELECT a.*,b.product_id, b.product_name, b.category, b.price, c.order_date, 
                    c.customer_id, d.full_name,c.total_amount 
            from order_items a
            left join products b
            ON a.product_id = b.product_id
            left join orders c
            ON a.order_id = c.order_id
            left join customers as d
            on c.customer_id = d.customer_id
    ),
    ordercount AS ( SELECT customer_id,full_name, 
            SUM(line_tOtal) AS total_revenue, 
            count(distinct order_item_id) ordercount
            FROM CTE
            GROUP BY customer_id,full_name
    )

    SELECT customer_id,full_name,total_revenue, 
            CONCAT ('Rank ' , DENSE_RANK() OVER (ORDER BY total_revenue DESC)) spend_rank
            FROM ordercount


![alt text](Images/Dense%20Rank.png)

### 8. List customers who placed more than 1 order and show customer_id, full_name, order_count, first_order_date, last_order_date.

    --List customers who placed more than 1 order and show customer_id, full_name, 
    --order_count, first_order_date, last_order_date.

    WITH CTE AS (
            SELECT a.*,b.product_id, b.product_name, b.category, b.price, c.order_date, 
                    c.customer_id, d.full_name,c.total_amount 
            from order_items a
            left join products b
            ON a.product_id = b.product_id
            left join orders c
            ON a.order_id = c.order_id
            left join customers as d
            on c.customer_id = d.customer_id
    ),
    ordercount AS (
            SELECT a.customer_id, a.full_name, count(distinct a.order_item_id) AS order_count, 
            min(a.order_date) first_order_date, max(b.order_date) last_order_date
            FROM CTE a
            INNER JOIN CTE b
            ON a.customer_id = b.customer_id
            GROUP  BY a.customer_id, a.full_name
        
    )

    SELECT * FROM ordercount
    WHERE order_count > 1

![alt text](Images/Customers%20who%20placed%20more%20than%201%20order.png)

### 9. Compute total loyalty points per customer. Include customers with 0 points.

    --Compute total loyalty points per customer. Include customers with 0 points.

    SELECT a.customer_id, a.full_name, 
        CASE WHEN SUM(points_earned) IS NULL 
            THEN 0 ELSE SUM(points_earned)
        END AS total_loyalty_points
    --SUM(points_earned) total_loyalty_points
    FROM CUSTOMERS a
    --Left join ensures all customers are returned even if there is no loyalty points for them.
    LEFT JOIN loyalty_points b
    ON a.customer_id = b.customer_id
    GROUP BY a.customer_id, a.full_name
    ORDER BY a.customer_id

![alt text](Images/Total%20loyalty%20points.png)

### 10. Assigning loyalty tiers based on total points.
The requirements allows that all total points greater than 500 should be refrred to as Gold, 100 to 499 falls under Silver tier and lesser than 100 is bronze.

    WITH CTE AS (
        SELECT a.customer_id, a.full_name, 
            CASE WHEN SUM(points_earned) IS NULL 
                THEN 0 ELSE SUM(points_earned)
            END AS tier_total_points
        --SUM(points_earned) total_loyalty_points
        FROM CUSTOMERS a
        --Left join ensures all customers are returned even if there is no loyalty points for them.
        LEFT JOIN loyalty_points b
        ON a.customer_id = b.customer_id
        GROUP BY a.customer_id, a.full_name
    ),	

    tiergroup AS (
        SELECT CTE.*, 
        CASE WHEN tier_total_points >= 500 THEN 'Gold'
            WHEN tier_total_points >= 100 AND tier_total_points < 500 THEN 'Silver'
            ELSE 'Bronze' END AS tier
        FROM CTE
        ),

    tiercount AS (
        SELECT customer_id, count(distinct loyalty_id) tier_count
        FROM loyalty_points
        GROUP BY customer_id
    ),
    tiers AS (
        SELECT a.*, tier_count
        FROM tiergroup a
        LEFT JOIN tiercount
        ON a.customer_id = tiercount.customer_id
    )

    SELECT customer_id, full_name,tier, tier_count, tier_total_points
    FROM tiers

![alt text](Images/Tier%20group.png)

### 11. Identify customers who spent more than ₦50,000 in total but have less than 200 loyalty points. Return customer_id, full_name, total_spend, total_points.

    --Identify customers who spent more than ₦50,000 in total but have less than 200 loyalty points. 
    --Return customer_id, full_name, total_spend, total_points

    WITH CTE AS (
        SELECT a.customer_id, a.full_name, 
            CASE WHEN SUM(points_earned) IS NULL 
                THEN 0 ELSE SUM(points_earned)
            END AS total_points,
            SUM(c.total_amount) total_spend
        FROM CUSTOMERS a
        --Left join ensures all customers are returned even if there is no loyalty points for them.
        LEFT JOIN loyalty_points b
        ON a.customer_id = b.customer_id
        LEFT JOIN orders c
        ON a.customer_id = c.customer_id
        GROUP BY a.customer_id, a.full_name
    ),	

    highcustomerlowloyalty AS (
        SELECT customer_id, full_name, total_spend, total_points
        FROM CTE
        WHERE total_spend > 50000 
        AND total_points < 200
        )
    SELECT * FROM highcustomerlowloyalty

![alt text](Images/High%20customer%20with%20low%20loyalty.png.png)

### 12. Flag customers as churn_risk if they have no orders in the last 90 days (relative to 2023-12-31) AND are in the Bronze tier. Return customer_id, full_name, last_order_date, total_points.

    --Flag customers as churn_risk if they have no orders in the last 90 days (relative to 2023-12-31) 
    --AND are in the Bronze tier. Return customer_id, full_name, last_order_date, total_points.

    WITH customerlastorderdate AS (
                    SELECT a.customer_id, a.full_name, MAX(b.order_date) last_order_date
                    FROM CUSTOMERS a
                    LEFT JOIN orders b
                    ON a.customer_id = b.customer_id
                    GROUP BY a.customer_id, a.full_name
    ),
    customersin90daysorder AS (
                    SELECT order_id, customer_id 
                    FROM orders
                    WHERE order_date >= '2023-12-31'::date - INTERVAL '90 days'			
    ),

    ordersnotinlast90days AS (
        SELECT * 
        FROM customerlastorderdate AS a
        WHERE NOT EXISTS (
                        SELECT order_id 
                        FROM customersin90daysorder 
                        WHERE customer_id = a.customer_id)
    ),

    pointearned AS (
        SELECT a.customer_id, a.full_name, 
            CASE WHEN SUM(points_earned) IS NULL 
                THEN 0 ELSE SUM(points_earned)
            END AS total_points
        FROM CUSTOMERS a
        LEFT JOIN loyalty_points b
        ON a.customer_id = b.customer_id
        GROUP BY a.customer_id, a.full_name
    ),	

    tiergroup AS (
        SELECT pointearned.*, 
        CASE WHEN total_points >= 500 THEN 'Gold'
            WHEN total_points >= 100 AND total_points < 500 THEN 'Silver'
            ELSE 'Bronze' END AS tier
        FROM pointearned
    )

    SELECT a.*, b.total_points
    FROM ordersnotinlast90days a
    LEFT JOIN tiergroup b
    ON a.customer_id =b.customer_id
    WHERE tier = 'Bronze'
    ORDER BY a.customer_id

![alt text](Images/Bronze%20customer%20with%20no%20order%20in%20last%2090%20days.png)

### Insights

**Customer Acquisition & Revenue Performance**

In 2023, LaunchMart acquired 15 new customers, demonstrating steady growth in its customer base. Revenue distribution shows a promising pattern with 2 high-value customers generating over ₦50,000 each, while 4 customers contributed below ₦10,000, indicating opportunities for revenue optimization.

**Top Performing Customers**

The **top 5 customers** by revenue performance (in descending order):

    1. Hannah Kim
    2. Jessica White
    3. Olivia Brown
    4. Brian Smith
    5. David Lee

**Monthly Revenue Trends**

July 2023 emerged as the peak revenue month with ₦59,000, closely followed by September (₦55,000). However, June (₦1,500) and October (₦6,000) showed significantly lower performance, suggesting potential seasonal fluctuations or execution challenges during these periods.

**Customer Engagement Analysis - Customer Activity Status (as of Dec 31, 2023):**

A total of 11 customers had no orders in the last 60 days as of December 31, 2023, while 2 customers — George Adewale and Emeka Nwosu — had no orders in the past 90 days. Both belong to the Bronze tier, with their last orders placed in April and June, respectively.

**Customer Value Metrics - Average Order Value (AOV) Leaders:**

In terms of Average Order Value (AOV), Jessica White ranked highest at ₦55,000, followed by Hannah Kim with ₦29,500.

**Customer Ranking System:**
Implemented dense ranking methodology for customers with at least one order, allowing for skipped rank numbers when multiple customers share identical performance metrics.

**Loyalty Program Performance- Tier Distribution Analysis:**

-- Bronze Tier: 5 customers (30% of customer base)

-- Silver Tier: 9 customers (60% - majority segment)

-- Gold Tier: 1 customer (6% - elite segment)

![alt text](Images/Tier%20Grouping%20Count.png)

**Key Observations & Opportunities**

Although Hannah Kim achieved the highest total spend (₦59,000), her loyalty points (190) fall below the recommended threshold of 200 points.

Chinwe Okafor recorded the highest number of orders (3), while 60% of customers placed 2 orders, and the remaining 33% placed only 1 order.

### Recommendations
1. Re-engage the 11 inactive customers, especially those with no orders in the last 60–90 days, through personalized emails or promotional offers.

2. Consider implementing automated reminders or reward-based incentives to encourage repeat purchases.

3. Review the loyalty points system to ensure it effectively rewards high-spending customers like Hannah Kim, whose total spend is high but loyalty points remain below the target threshold.

4. Introduce tier-based benefits to motivate Bronze-tier customers to advance to Silver or Gold.

5. Establish dashboards to track order frequency, AOV, and loyalty points in real time, enabling proactive interventions before customer inactivity extends beyond 60 days.