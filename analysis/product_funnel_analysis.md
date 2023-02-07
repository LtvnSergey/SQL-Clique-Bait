# Project: Clique-Bait

# Digital Analysis

## Questions

  1. [How many times was each product viewed?](#1-how-many-times-was-each-product-viewed)
  2. [How many times was each product added to cart?](#2-how-many-times-was-each-product-added-to-cart)
  3. [How many times was each product added to a cart but not purchased (abandoned)?](#3-how-many-times-was-each-product-added-to-a-cart-but-not-purchased-(abandoned))
  4. [How many times was each product purchased?](#4.how-many-times-was-each-product-purchased)
  5. [Which product had the most views, cart adds and purchases?](#5-which-product-had-the-most-views,-cart-adds-and-purchases)
  6. [Which product was most likely to be abandoned?](#6-which-product-was-most-likely-to-be-abandoned)
  7. [Which product had the highest view to purchase percentage?](#7-which-product-had-the-highest-view-to-purchase-percentage)
  8. [What is the average conversion rate from view to cart add?](#8-what-is-the-average-conversion-rate-from-view-to-cart-add)
  9. [What is the average conversion rate from cart add to purchase?](#9-what-is-the-average-conversion-rate-from-cart-add-to-purchase)


### 1. How many times was each product viewed?
````sql 

WITH events_products AS (
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
	
	add_cart AS (
	SELECT page_name, COUNT(*) AS add_cart
	FROM events_products
	WHERE event_type IN 
		(SELECT event_type FROM clique_bait.event_identifier 
		 WHERE event_name='Add to Cart')
	GROUP BY page_name
	),
	
	abandoned AS (
	SELECT page_name, COUNT(*) AS abandoned
	FROM events_products
	WHERE event_type IN 
		(SELECT event_type FROM clique_bait.event_identifier 
		 WHERE event_name='Add to Cart')
	AND
		visit_id NOT IN 
		(SELECT DISTINCT(visit_id) FROM events_products
		 WHERE event_type IN 
			(SELECT event_type FROM clique_bait.event_identifier 
		 	 WHERE event_name='Purchase'))	
	GROUP BY page_name
	),
	
	purchased AS (
	SELECT page_name, COUNT(*) AS purchased
	FROM events_products
	WHERE visit_id IN 
		(SELECT DISTINCT(visit_id) FROM clique_bait.events
		 WHERE event_type IN 
			(SELECT event_type FROM clique_bait.event_identifier 
		 	 WHERE event_name='Purchase'))	
	GROUP BY page_name
	),
	
	combined AS (
	SELECT viewed.page_name, viewed.page_view, add_cart.add_cart, abandoned.abandoned, purchased.purchased
	FROM viewed 
	JOIN add_cart ON viewed.page_name = add_cart.page_name
	JOIN abandoned ON add_cart.page_name = abandoned.page_name
	JOIN purchased ON abandoned.page_name = purchased.page_name
	)
	
SELECT *
FROM combined
````

![image](https://user-images.githubusercontent.com/35038779/217338052-1dab0c30-c236-4fa5-b301-41a1f260179c.png)


### 2. How many times was each product added to cart?


### 3. How many times was each product added to a cart but not purchased (abandoned)?


### 4. How many times was each product purchased?

### 5. Which product had the most views, cart adds and purchases?


### 6. Which product was most likely to be abandoned?

### 7. Which product had the highest view to purchase percentage?

### 8. What is the average conversion rate from view to cart add?

### 9. What is the average conversion rate from cart add to purchase?
