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


### 4. What is the number of events for each event type?


### 5. What is the percentage of visits which have a purchase event?


### 6. What is the percentage of visits which view the checkout page but do not have a purchase event?


### 7. What are the top 3 pages by number of views?


### 8. What is the number of views and cart adds for each product category?


### 9. What are the top 3 products by purchases?
