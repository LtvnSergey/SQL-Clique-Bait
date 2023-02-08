# Project: Clique-Bait

# Product Funnel Analysis

## Questions

  1. [How many times was each product: viewed, aded to cart, abandoned, purchased?](#1-how-many-times-was-each-product-viewed-aded-to-cart-abandoned-purchased)
  2. [Which product had the most views, cart adds, purchases, most likely to be abandoned?](#2-which-product-had-the-most-views-cart-adds-purchases-most-likely-to-be-abandoned)
  3. [Which product had the highest view to purchase percentage?](#3-which-product-had-the-highest-view-to-purchase-percentage)
  4. [What is the average conversion rate from view to cart add?](#4-what-is-the-average-conversion-rate-from-view-to-cart-add)
  5. [What is the average conversion rate from cart add to purchase?](#5-what-is-the-average-conversion-rate-from-cart-add-to-purchase)


### 1. How many times was each product: viewed, aded to cart, abandoned, purchased?

````sql 
	/* Table with page name, visit_id and event_type */
WITH 	events_products AS (
		SELECT ph.page_name, e.visit_id, e.event_type
		FROM clique_bait.events AS e
		JOIN
		clique_bait.page_hierarchy AS ph
		ON e.page_id = ph.page_id
		WHERE ph.product_category IS NOT NULL),

		viewed AS (
			SELECT page_name, COUNT(*) AS page_view
			FROM events_products
			WHERE event_type IN 
				(SELECT event_type FROM clique_bait.event_identifier 
				 WHERE event_name='Page View')
		GROUP BY page_name
	),
	
	/* Table with products added to cart */
	add_cart AS (
		SELECT page_name, COUNT(*) AS add_cart
		FROM events_products
		WHERE event_type IN 
			(SELECT event_type FROM clique_bait.event_identifier 
			 WHERE event_name='Add to Cart')
		GROUP BY page_name
	),
	
	/* Table with products that are abandoned */
	abandoned AS (
		SELECT page_name, COUNT(*) AS abandoned
		FROM events_products
		WHERE event_type IN 
			(SELECT event_type FROM clique_bait.event_identifier 
			 WHERE event_name='Add to Cart')
		AND
			visit_id NOT IN 
			(SELECT DISTINCT(visit_id) FROM clique_bait.events
			 WHERE event_type IN 
				(SELECT event_type FROM clique_bait.event_identifier 
				 WHERE event_name='Purchase'))	
		GROUP BY page_name
	),
	
	/* Table with pirchased products */
	purchased AS (
		SELECT page_name, COUNT(*) AS purchased
		FROM events_products

		WHERE event_type IN 
			(SELECT event_type FROM clique_bait.event_identifier 
			 WHERE event_name='Add to Cart')
		AND visit_id IN 
			(SELECT DISTINCT(visit_id) FROM clique_bait.events
			 WHERE event_type IN 
				(SELECT event_type FROM clique_bait.event_identifier 
				 WHERE event_name='Purchase'))	
		GROUP BY page_name
	),
	
	/* Combine tables with viewed, added to cart, abandoned, purchased products */
	product_summary AS (
		SELECT viewed.page_name, viewed.page_view, add_cart.add_cart, abandoned.abandoned, purchased.purchased
		FROM viewed 
		JOIN add_cart ON viewed.page_name = add_cart.page_name
		JOIN abandoned ON add_cart.page_name = abandoned.page_name
		JOIN purchased ON abandoned.page_name = purchased.page_name
	)
	
SELECT *
FROM product_summary
````

![image](https://user-images.githubusercontent.com/35038779/217620673-b53e2e4c-9e73-4ab6-a7bf-3af9f3d44488.png)


### 2. Which product had the most views, cart adds, purchases, most likely to be abandoned?

````sql
/* Product with most views */
SELECT page_name, page_view
FROM product_summary
ORDER BY page_view DESC
LIMIT 1
````
![image](https://user-images.githubusercontent.com/35038779/217539829-e7476afc-9cd9-4055-b3a6-36ba31de291b.png)


````sql
/* Product with most cart adds */
SELECT page_name, add_cart
FROM product_summary
ORDER BY add_cart DESC
LIMIT 1
````
![image](https://user-images.githubusercontent.com/35038779/217540059-fbf94811-dbf7-42a3-9a0c-9759eacdfb72.png)

````sql
/* Product with most purchases */
SELECT page_name, purchased
FROM product_summary
ORDER BY purchased DESC
LIMIT 1
````
![image](https://user-images.githubusercontent.com/35038779/217620853-b9c4e145-dc2f-459a-953e-22ce34c7229f.png)


````sql
/* Product most likely to be adandoned */
SELECT page_name, abandoned
FROM product_summary
ORDER BY abandoned DESC
LIMIT 1
````
![image](https://user-images.githubusercontent.com/35038779/217545293-e64a4bd3-6290-4a4b-b006-78f45d37c7c4.png)

* Product with most views: Oyster
* Product with most cart adds: Lobster
* Product with most purchases: Lobster
* Product most likely to be adandoned: Russian Caviar


### 3. Which product had the highest view to purchase percentage?

````sql
SELECT page_name, 100 * purchased/page_view AS purchase_per_view_percentage
FROM product_summary
ORDER BY purchase_per_view_percentage DESC
LIMIT 1
````

![image](https://user-images.githubusercontent.com/35038779/217621035-d1bdcb5a-8574-4bbd-a0c2-a49278bc011b.png)


### 4. What is the average conversion rate from view to cart add?

````sql
SELECT ROUND(AVG(100*add_cart::decimal/page_view::decimal), 2) AS avg_add_cart_per_view_conversion
FROM product_summary
````
![image](https://user-images.githubusercontent.com/35038779/217623168-e8c58238-dc3f-46f7-8852-8bd136003866.png)


### 5. What is the average conversion rate from cart add to purchase?

````sql
SELECT ROUND(AVG(100*purchased::decimal/add_cart::decimal), 2) AS avg_purchase_per_add_cart
FROM product_summary
````


![image](https://user-images.githubusercontent.com/35038779/217623449-df58829b-aa1f-444a-8834-181088903a46.png)

