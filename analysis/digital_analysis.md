# Project: Clique-Bait

# Digital Analysis

## Questions

  1. [How many users are there?](#1-how-many-users-are-there)
  2. [How many cookies does each user have on average?](#2-how-many-cookies-does-each-user-have-on-average)
  3. [What is the unique number of visits by all users per month?](#3-what-is-the-unique-number-of-visits-by-all-users-per-month)
  4. [What is the number of events for each event type?](#4-what-is-the-number-of-events-for-each-event-type)
  5. [What is the percentage of visits which have a purchase event?](#5-what-is-the-percentage-of-visits-which-have-a-purchase-event)
  6. [What is the percentage of visits which view the checkout page but do not have a purchase event?](#6-what-is-the-percentage-of-visits-which-view-the-checkout-page-but-do-not-have-a-purchase-event)
  7. [What are the top 3 pages by number of views?](#7-what-are-the-top-3-pages-by-number-of-views)
  8. [What is the number of views and cart adds for each product category?](#8-what-is-the-number-of-views-and-cart-adds-for-each-product-category)
  9. [What are the top 3 products by purchases?](#9-what-are-the-top-3-products-by-purchases)

### 1. How many users are there

````sql
SELECT COUNT(DISTINCT USER_ID) AS USERS_AMOUNT
FROM CLIQUE_BAIT.USERS
````

![image](https://user-images.githubusercontent.com/35038779/217037618-b3136582-8402-4ea0-86dd-e7cccfd840a3.png)

* There are 500 users

### 2. How many cookies does each user have on average?

````sql
SELECT 
 ROUND(AVG(AMOUNT),0) AS AVERAGE_COOKIE_PER_USER
FROM
 (SELECT COUNT(*) AS AMOUNT
  FROM CLIQUE_BAIT.USERS
  GROUP BY USER_ID) AS COUNT_COOKIES
````

![image](https://user-images.githubusercontent.com/35038779/217040456-6adaf38f-f61a-482c-9065-e4900ba5b89d.png)

* There are 4 cookies per user on average



### 3. What is the unique number of visits by all users per month?

````sql
SELECT 
 EXTRACT(MONTH FROM EVENT_TIME) AS month,
 COUNT(DISTINCT(VISIT_ID)) AS NUMBER_OF_VISITS
FROM CLIQUE_BAIT.EVENTS
GROUP BY EXTRACT(MONTH FROM EVENT_TIME)
````

![image](https://user-images.githubusercontent.com/35038779/217043407-2966f15f-bcfb-4adc-bef2-52288ae3cac7.png)



### 4. What is the number of events for each event type?

````sql
SELECT 
 EVENT_TYPE AS EVENT_TYPE,
 COUNT(*) AS NUMBER_OF_EVENT_TYPES
FROM CLIQUE_BAIT.EVENTS
GROUP BY EVENT_TYPE
ORDER BY EVENT_TYPE
````

![image](https://user-images.githubusercontent.com/35038779/217044208-cb4fe60c-62b8-4139-8176-4611b9609154.png)



### 5. What is the percentage of visits which have a purchase event?

````sql
SELECT 100 * COUNT(DISTINCT(e.visit_id)) /
	(SELECT COUNT(DISTINCT(visit_id)) FROM CLIQUE_BAIT.EVENTS) AS purchase_percent
FROM CLIQUE_BAIT.EVENTS AS e
LEFT JOIN 
CLIQUE_BAIT.EVENT_IDENTIFIER AS ei
ON e.event_type = ei.event_type
WHERE ei.event_name='Purchase'
````

![image](https://user-images.githubusercontent.com/35038779/217049753-44813180-71fb-4faa-844b-c84252ac5305.png)

* Percentage of purchase event visits equals 49 %

### 6. What is the percentage of visits which view the checkout page but do not have a purchase event?

````sql
WITH checkout_purchase AS (
SELECT 
  visit_id,
  MAX(CASE WHEN event_type = 1 AND page_id = 12 THEN 1 ELSE 0 END) AS checkout,
  MAX(CASE WHEN event_type = 3 THEN 1 ELSE 0 END) AS purchase
FROM clique_bait.events
GROUP BY visit_id)

SELECT 
  ROUND(100 * (1-(SUM(purchase)::numeric/SUM(checkout))),2) AS percentage_checkout_view_with_no_purchase
FROM checkout_purchase
````

![image](https://user-images.githubusercontent.com/35038779/217055633-c5040d28-59a6-4c7c-af6a-9201dd5dfd32.png)

* Percentage of of visits with page checkout but without purchase equals 15.5 %


### 7. What are the top 3 pages by number of views?


````sql
SELECT 
  ph.page_name, 
  COUNT(*) AS page_views
FROM clique_bait.events AS e
JOIN clique_bait.page_hierarchy AS ph
  ON e.page_id = ph.page_id
WHERE e.event_type = 1 
GROUP BY ph.page_name
ORDER BY page_views DESC 
LIMIT 3
````

![image](https://user-images.githubusercontent.com/35038779/217057193-3a1ff64a-b96b-4eab-87e0-49d6fdac7fee.png)


### 8. What is the number of views and cart adds for each product category?

````sql
SELECT 
  ph.product_category, 
  SUM(CASE WHEN e.event_type = 1 THEN 1 ELSE 0 END) AS page_views,
  SUM(CASE WHEN e.event_type = 2 THEN 1 ELSE 0 END) AS cart_adds
FROM clique_bait.events AS e
JOIN clique_bait.page_hierarchy AS ph
  ON e.page_id = ph.page_id
WHERE ph.product_category IS NOT NULL
GROUP BY ph.product_category
ORDER BY page_views DESC
````

![image](https://user-images.githubusercontent.com/35038779/217316337-62595d19-df94-49ba-8828-64259a1d3148.png)


### 9. What are the top 3 products by purchases?

````sql
with ordered_rows AS(
    SELECT
      page_name,
      event_name,
      COUNT(event_name) AS number_of_purchases,
      ROW_NUMBER() OVER (
        ORDER BY
          COUNT(event_name) DESC
      ) AS row
    FROM
      events AS e
      JOIN page_hierarchy AS pe ON e.page_id = pe.page_id
      JOIN event_identifier AS ei ON e.event_type = ei.event_type
    WHERE
      visit_id in (
        SELECT
          distinct visit_id
        FROM
          events AS ee
        WHERE
          event_type = 3
      )
      AND product_id > 0
      AND event_name = 'Add to Cart'
    GROUP BY
      1,
      2
  )
SELECT
  page_name,
  number_of_purchases
FROM
  ordered_rows
WHERE
  row in (1, 2, 3)
````

![image](https://user-images.githubusercontent.com/35038779/217320804-032fdfca-22f4-4105-89f8-357d6105c7f2.png)





