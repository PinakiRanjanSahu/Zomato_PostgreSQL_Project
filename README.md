# Zomato Data Analysis using SQL

# Creating Customers Table

    CREATE TABLE customers
    (
         customer_id INT PRIMARY KEY,
         customer_name VARCHAR(25),
         reg_date DATE
     );  

# Creating Restaurants Table

    CREATE TABLE restaurants
    (
        restaurant_id INT PRIMARY KEY,
        restaurant_name VARCHAR(55),
        city VARCHAR(15),
        opening_hours VARCHAR(55)
    );

# Creating Orders Table
    CREATE TABLE orders
    (
        order_id INT PRIMARY KEY,
        customer_id INT,        -- coming from customers table
        restaurant_id INT,      -- coming from restaurants table
        order_item VARCHAR(100),
        order_date DATE,
        order_time TIME,
        order_status VARCHAR(100),
        total_amount DECIMAL(10,2)
    );


# adding constraints fk_customers
    ALTER TABLE orders
    ADD CONSTRAINT fk_customers
    FOREIGN KEY (customer_id)
    REFERENCES customers(customer_id);

# adding constraints fk_restaurent
    ALTER TABLE orders
    ADD CONSTRAINT fk_restaurants
    FOREIGN KEY (restaurant_id)
    REFERENCES restaurants(restaurant_id);

# Creating Riders Table
    CREATE TABLE riders
    (
        rider_id INT PRIMARY KEY,
        rider_name VARCHAR(55),
        sign_up DATE
    );
# Creating Deliveries Table
    
    CREATE TABLE deliveries
    (
        delivery_id INT PRIMARY KEY,
        order_id INT,           -- coming from orders table
        delivery_status VARCHAR(35),
        delivery_time TIME,
        rider_id INT,-- coming from riders table
    	CONSTRAINT fk_orders FOREIGN KEY (order_id) REFERENCES orders(order_id),
    	CONSTRAINT fk_riders FOREIGN KEY (rider_id) REFERENCES riders(rider_id)
    );  

# --end of Schemas--------------------------------------------------------------------


#Explore EDA

		----------------------------------------------------
# IMPORT DATASET
		----------------------------------------------------
		SELECT * FROM customers;
		SELECT * FROM restaurants;
        SELECT * FROM orders;
		SELECT * FROM deliveries;
		SELECT * FROM riders;
		----------------------------------------------------
# Handling NULL Values
		----------------------------------------------------
		SELECT COUNT(*) FROM restaurants
		WHERE 
		  restaurant_name IS NULL
		  OR city IS NULL
		  OR opening_hours IS NULL
		  
		SELECT COUNT(*) FROM restaurants
		WHERE 
		  restaurant_name IS NULL
		  OR city IS NULL
		  OR opening_hours IS NULL;
		
		SELECT COUNT(*) FROM orders
		WHERE 
		  order_item IS NULL
		  OR order_date IS NULL
		  OR order_time IS NULL
		  OR order_status IS NULL
		  OR total_amount IS NULL;
		

# Analysis & Report
		
# Q.1
# Write a query to find the top 5 most frequently ordered dishes by customer called "Kevin" in the last 1 year
--
		
-- join cx and orders
--filter the data for last one year
--filter for 'kevin'
-- group by cx_id,dishes,cnt
		
		
    SELECT 
    customer_name,
    total_orders,
    dishes
    FROM
    (SELECT 
    c.customer_id,
    c.customer_name,
    o.order_item as dishes,
    COUNT(*) as total_orders,
    DENSE_RANK() OVER (ORDER BY COUNT(*) DESC) as RANK
    FROM orders as o 
    JOIN
    	customers as c
    ON c.customer_id=o.order_id
    WHERE
    	o.order_date>= CURRENT_DATE - INTERVAL '1 YEAR'
    AND
    	c.customer_name='Saksham Ben'
    GROUP BY 1,2,3
    ORDER BY 1,4 DESC) as t1
    WHERE RANK <= 5
		
		
		
# 2. Popular Time slot
# Q.Identify the tiem slots during which the most orders are placed, based on 2 hour intervals.
--
		

# 1ST Approch

		
	SELECT
	CASE
		WHEN EXTRACT(HOUR FROM order_time) BETWEEN 0 AND 1 THEN '00:00 - 02:00'
		WHEN EXTRACT(HOUR FROM order_time) BETWEEN 2 AND 3 THEN '02:00 - 04:00'
		WHEN EXTRACT(HOUR FROM order_time) BETWEEN 4 AND 5 THEN '04:00 - 06:00'
		WHEN EXTRACT(HOUR FROM order_time) BETWEEN 6 AND 7 THEN '06:00 - 08:00'
		WHEN EXTRACT(HOUR FROM order_time) BETWEEN 8 AND 9 THEN '08:00 - 10:00'
		WHEN EXTRACT(HOUR FROM order_time) BETWEEN 10 AND 11 THEN '10:00 - 12:00'
		WHEN EXTRACT(HOUR FROM order_time) BETWEEN 12 AND 13 THEN '12:00 - 14:00'
		WHEN EXTRACT(HOUR FROM order_time) BETWEEN 14 AND 15 THEN '14:00 - 16:00'
		WHEN EXTRACT(HOUR FROM order_time) BETWEEN 16 AND 17 THEN '16:00 - 18:00'
		WHEN EXTRACT(HOUR FROM order_time) BETWEEN 18 AND 19 THEN '18:00 - 20:00'
		WHEN EXTRACT(HOUR FROM order_time) BETWEEN 20 AND 21 THEN '20:00 - 22:00'
		WHEN EXTRACT(HOUR FROM order_time) BETWEEN 22 AND 23 THEN '22:00 - 0:00'
	END as time_slot,
	COUNT(order_id) as order_count
	FROM orders
	GROUP BY time_slot
	ORDER BY order_count
		
		-- select 00:59:59AM--0
		-- select 01:59:59AM--1

		
		
		
  # 2nd Approch using FLOOR
		
		SELECT
			FLOOR(EXTRACT(HOUR FROM order_time)/2)*2 as start_time,
			FLOOR(EXTRACT(HOUR FROM order_time)/2)*2+2 as end_time,
			COUNT(*) as total_orders
		FROM orders
		GROUP BY 1,2
		ORDER BY 3 DESC
		
		--23:22PM /2 -- 11 * 2 = 22 START,
		--23:22PM /2 -- 11 * 2 = 22 +2 = 24 END

		

		



# 3. Order Value Analysis
# Question: Find the average order value per customer who has placed more than 90 orders.
-- Return customer_name and aov(average order value)

		
		
    SELECT 
    c.customer_name,
    AVG(o.total_amount) as average_order_value
    FROM orders as o
    JOIN customers as c
    ON c.customer_id = o.customer_id
    GROUP BY 1
    HAVING COUNT(order_id)>90
    --ORDER BY average_order_value DESC;


		
# 4.High Value Customers
# Question: List the customers who have spent more than 70K in total on food orders
-- return customer_name, and customer_id!

      SELECT 
      	c.customer_name,
      	SUM(o.total_amount) as total_spent
      FROM orders as o
      	JOIN customers as c
      	ON c.customer_id = o.customer_id
      GROUP BY 1
      HAVING sum(total_amount)>70000
      ORDER BY total_spent DESC;
      	



# 5. Orders without Delivery
# Question: Write a query to find orders that were placed but not delivered.
-- Return each restaurant name city and number of not delivered orders.

#1st approch

    SELECT*
    FROM orders as o
    LEFT JOIN
    restaurants as r
    ON r.restaurant_id = o.restaurant_id
    LEFT JOIN 
    deliveries as d
    ON d.order_id = o.order_id
    WHERE d.delivery_status = 'Not Fullfilled';




# 2nd Approch

    SELECT
    	r.restaurant_name,
    	COUNT(o.order_id) as cnt_not_delivered_orders
    	
    FROM orders as o
    LEFT JOIN
    restaurants as r
    ON r.restaurant_id = o.restaurant_id
    LEFT JOIN
    deliveries as d
    ON d.delivery_id = o.order_id
    WHERE d.delivery_id IS NULL
    GROUP BY 1


# 6.Restaurant Revenue Rnaking
# Q rank Restaurants by their total revenue from last year, including their name.
-- total revenue, and rank within their city

    WITH ranking_table
    as(
    SELECT 
    r.city,
    r.restaurant_name,
    SUM(o.total_amount) as total_revenue,
    RANK() OVER(PARTITION BY r.city ORDER BY SUM(o.total_amount) DESC) as RANK
    FROM orders as o
    JOIN restaurants as r
    ON r.restaurant_id = o.restaurant_id
    WHERE o.order_date >= CURRENT_DATE - INTERVAL '1 year'
    GROUP BY 1,2
    )
    SELECT *
    FROM ranking_table
    WHERE RANK = 1




# 7.Most Popular Dish By city
# Identify the most popular dish in each city based on the number of orders


    WITH rankdb
    as(
    SELECT
    	o.order_item as dish,
    	COUNT(o.order_id) as total_orders,
    	r.city,
    RANK() OVER(PARTITION BY r.city ORDER BY COUNT(o.order_id)DESC) as RANK
    FROM orders as o
    JOIN restaurants as r
    ON r.restaurant_id = o.restaurant_id
    GROUP BY 1,3)
    SELECT * FROM rankdb
    WHERE RANK = 1





# 8. Customers churn
# Q. Find customers who haven't placed any order in 2024 but did in 2023


    
    SELECT 
    DISTINCT customer_id
    FROM orders
    WHERE
    	EXTRACT(YEAR FROM order_date)= 2023
    	AND
    	customer_id NOT IN
    	(SELECT DISTINCT customer_id FROM orders
    	WHERE EXTRACT(YEAR FROM order_date)= 2023)






# 9.Cancellation Rate Comparison:
# Calculate and compare the order concelation rate for each restaurant between the
-- current year and the previous year


    SELECT 
    restaurant_id,
    COUNT(o.order_id) as total_orders,
    COUNT(CASE WHEN d.delivery_id IS NULL THEN 1 END) not_delivered
    FROM orders as o
    LEFT JOIN deliveries as d
    ON o.order_id = d.order_id
    WHERE EXTRACT(YEAR FROM order_date)=2023
    GROUP BY 1



# 10. Rider Average Delivery Time:
# Q. Determine each Rider's average delivery time.

    SELECT 
    	o.order_id,
    	rider_id,
    	o.order_time,
    	d.delivery_time,
    	o.order_time - d.delivery_time as time_difference,
    	EXTRACT(EPOCH FROM(d.delivery_time - o.order_time + CASE 
    	WHEN d.delivery_time>o.order_time 
    	THEN INTERVAL '1 day' 
    	ELSE INTERVAL '0 day' END ))/60 as time_diff_inseconds
    FROM orders as o 
    JOIN deliveries as d 
    ON o.order_id = d.order_id
    WHERE d.delivery_status = 'Completed';




# 11. Monthly Grow Ratio:
# Q. Calculate each restaurant's growth ratio based on total number of delivered orders since joining
--
-- LAST MONTH-20,CURRENT MONTH-30  (CS - LS)/LS *100


    WITH growth_ratio 
    as(
    SELECT 
    o.restaurant_id,
    TO_CHAR(o.order_date,'MM-YY') as Month,
    COUNT(o.order_id) as No_of_orders,
    LAG(COUNT(o.order_id),1) OVER(PARTITION BY o.restaurant_id ORDER BY TO_CHAR(o.order_date,'MM-YY')) as prev_m_orders-- window function
    FROM orders as o
    JOIN deliveries as d
    ON o.order_id=d.order_id
    WHERE d.delivery_status = 'Completed'
    GROUP BY 1,2
    ORDER BY 1,2
    )
    SELECT 
    restaurant_id,
    Month,
    No_of_orders,
    (No_of_orders::numeric-prev_m_orders::numeric)/prev_m_orders * 100 as GROWTH_RATIO
    FROM growth_ratio





# 12. Customer Segmentation
# Q.Customer Segmentation: Segment customers into ' Gold' or 'Silver' groups based ont their total spending 
-- compared to the average order value(AOV). If the customers total spending exceed aov, 
-- label them as 'Gold'; otherwise label them as 'Silver'. Write an SQL query to determine each segment's
-- total number of orders and total revenue
--cx total spend
--AOV
--gold or silver
-- each category and total orders and total revenue

    WITH customer_label
    as
    (
    SELECT 
    customer_id,
    COUNT(order_id) as total_orders,
    SUM(total_amount) as total_spent,
    CASE WHEN SUM(total_amount)> 30000 THEN 'GOLD' ELSE 'SILVER' END as cx_category
    FROM orders
    GROUP BY 1
    ORDER BY 1
    )
    SELECT 
    cx_category,
    SUM(total_orders) as total_orders,
    SUM(total_spent) as total_spent
    FROM customer_label
    GROUP BY 1







# 13. Rider Monthly Earning:
# Q.Calculate each rider's monthly earning, assuming they earn 8% of the order_amount
--


    SELECT
    d.rider_id,
    TO_CHAR(o.order_date,'MM-YY') as Month,
    SUM(o.total_amount) as total_order_amount,
    SUM(o.total_amount)*0.08 as rider_earnings
    FROM orders as o
    JOIN deliveries as d
    ON o.order_id = d.order_id
    GROUP BY 1,2
    ORDER BY 1,2,3



# 14.Rider Rating Analysis
# Q. Find the number of 5star, 4star,3star ratings each rider has.
-- riders receive this based on delivery time
-- if they deliver in less then 15 min they get 5 star rating
-- if the deliver 15 - 30 min they get 4 star ratings
-- if they deliver after 30 min they get 3 star rating.

    SELECT 
    rider_id,
    rating,
    COUNT(*) as number_of_ratings 
    FROM
    (
    WITH delivery_book
    as
    (
    SELECT 
    d.rider_id,
    o.order_id,
    d.delivery_time,
    o.order_time,
    EXTRACT(EPOCH FROM(d.delivery_time - o.order_time + 
    CASE WHEN d.delivery_time < o.order_time 
    THEN INTERVAL '1 DAY' 
    ELSE '0 DAY' 
    END))/60 as delivery_took_time
    FROM orders as o
    JOIN 
    deliveries as d
    ON o.order_id = d.order_id
    )
    SELECT
    rider_id,
    delivery_took_time,
    CASE
    WHEN delivery_took_time < 15 THEN '5 STAR'
    WHEN delivery_took_time BETWEEN 15 AND 30 THEN '4 STAR'
    ELSE '3 STAR'
    END as rating
    FROM delivery_book
    )as t2
    
    GROUP BY 1,2
    
    



# 15. Order Frequency By Day
#  Q.Analyze order frequency per day of the week and identify the peak day ofr each restaurant.
--

    WITH frequency
    as
    (SELECT 
    r.restaurant_name,
    COUNT(o.order_id) as Orders,
    TO_CHAR(o.order_date,'Day') as Day,
    RANK() OVER(PARTITION BY r.restaurant_name ORDER BY COUNT(o.order_id)DESC) as Rank
    --o.order_date
    FROM orders as o
    JOIN restaurants as r
    ON r.restaurant_id = o.restaurant_id
    GROUP BY 1,3
    )
    SELECT 
    restaurant_name,
    Orders,
    Day,
    Rank
    FROM frequency
    WHERE Rank = 1  




# 16.Customer Lifeltime Value (CLV)
# Q. Calculate the total revenue generated by each customer over all orders.
--

    SELECT 
    o.customer_id,
    c.customer_name,
    COUNT(o.order_id) as total_orders,
    SUM(total_amount) as CLV
    FROM orders as o
    JOIN customers as c
    ON c.customer_id = o.customer_id
    GROUP BY 1,2



# 17.Monthly sales trends:
# Q.Identify trends by comparing each month's total sales to previous month

    SELECT 
    EXTRACT(YEAR FROM order_date) as year,
    EXTRACT(MONTH FROM order_date) as month,
    SUM(total_amount) as revenue,
    LAG(SUM(total_amount),1) OVER(ORDER BY EXTRACT(YEAR FROM order_date), EXTRACT(MONTH FROM order_date)) as prev_month
    FROM orders
    GROUP BY 1,2




# 18. Rider Efficiency:
# Evaluate rider efficiency by dertimining average items and identifying those with the lowest ans highest average.
--


    WITH deliver
    as
    (
    SELECT 
    d.rider_id,
    EXTRACT(EPOCH FROM(d.delivery_time - o.order_time + CASE 
    	WHEN d.delivery_time>o.order_time 
    	THEN INTERVAL '1 day' 
    	ELSE INTERVAL '0 day' END ))/60 as time_delivered
    FROM orders as o
    JOIN deliveries as d
    ON o.order_id = d.order_id
    WHERE d.delivery_status = 'Completed'
    GROUP BY 1,2
    ),
    rider_time AS
    (
    SELECT
    rider_id,
    AVG(time_delivered)/60 as delivery_time 
    FROM deliver
    GROUP BY 1
    )
    SELECT 
    MIN(delivery_time) as min_time_in_min,
    MAX(delivery_time) as max_time_in_min
    FROM rider_time



# 19. Order Item Popularity:
# Track the po[pularity of the specific order items over time ad identify seasonal demand spikes.
--

    SELECT 
    product,
    seasons,
    frequency
    FROM
    (
    SELECT 
    order_item as product,
    EXTRACT(MONTH FROM order_date) as Month,
    CASE
    	WHEN EXTRACT(MONTH FROM order_date) BETWEEN 4 AND 6 THEN 'SPRING'
    	WHEN EXTRACT(MONTH FROM order_date) > 6 AND EXTRACT(MONTH FROM order_date) < 9 THEN 'SUMMER'
    	ELSE 'WINTER' END AS seasons,
    COUNT(order_id) as frequency
    FROM orders
    GROUP BY 1,2
    ) as t1
    GROUP BY 1,2,3




# 20. Rank each city based on the total revenue for last year 2023
--

    SELECT 
    r.restaurant_id,
    r.restaurant_name,
    RANK() OVER(ORDER BY SUM(total_amount) DESC)  as Rank,
    SUM(total_amount) as total_revenue
    FROM orders as o
    JOIN restaurants as r
    ON o.restaurant_id = r.restaurant_id
    GROUP BY 1,2



# END Of reports


