# Project: Clique-Bait

# Campaigns Analysis

## Problem

Generate a table that has 1 single row for every unique visit_id record and has the following columns:
- `user_id`
- `visit_id`
- `visit_start_time`: the earliest event_time for each visit
- `page_views`: count of page views for each visit
- `cart_adds`: count of product cart add events for each visit
- `purchase`: 1/0 flag if a purchase event exists for each visit
- `campaign_name`: map the visit to a campaign if the visit_start_time falls between the start_date and end_date
- `impression`: count of ad impressions for each visit
- `click`: count of ad clicks for each visit
- (Optional column) `cart_products`: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the sequence_number)
  

````sql
WITH 	
    product_campaing_summary AS (
	SELECT 
	  u.user_id, e.visit_id, 
	  MIN(e.event_time) AS visit_start_time,
	  c.campaign_name,
	  SUM(CASE WHEN e.event_type IN (SELECT event_type FROM clique_bait.event_identifier 
					 WHERE event_name='Page View') THEN 1 ELSE 0 END) AS page_views,
	  SUM(CASE WHEN e.event_type IN (SELECT event_type FROM clique_bait.event_identifier 
					 WHERE event_name='Add to Cart') THEN 1 ELSE 0 END) AS cart_adds,
	  SUM(CASE WHEN e.event_type IN (SELECT event_type FROM clique_bait.event_identifier 
					 WHERE event_name='Purchase') THEN 1 ELSE 0 END) AS purchase,
	  SUM(CASE WHEN e.event_type IN (SELECT event_type FROM clique_bait.event_identifier 
					 WHERE event_name='Ad Impression') THEN 1 ELSE 0 END) AS impression, 
	  SUM(CASE WHEN e.event_type IN (SELECT event_type FROM clique_bait.event_identifier 
					 WHERE event_name='Ad Click') THEN 1 ELSE 0 END) AS click, 
	  STRING_AGG(CASE WHEN p.product_id IS NOT NULL AND e.event_type = 2 THEN p.page_name ELSE NULL END, 
		', ' ORDER BY e.sequence_number) AS cart_products
	FROM clique_bait.users AS u
	INNER JOIN clique_bait.events AS e
	  ON u.cookie_id = e.cookie_id
	LEFT JOIN clique_bait.campaign_identifier AS c
	  ON e.event_time BETWEEN c.start_date AND c.end_date
	LEFT JOIN clique_bait.page_hierarchy AS p
	  ON e.page_id = p.page_id
	GROUP BY u.user_id, e.visit_id, c.campaign_name
	)
			
SELECT * 
FROM product_campaing_summary
````

![image](https://user-images.githubusercontent.com/35038779/217762488-711b41f7-2782-4de2-b3de-60a16bde2e4a.png)




````sql
/* Summarize metrics for each user during different campagns */
WITH user_summary AS (
	SELECT 
		campaign_name,	
		user_id,
		SUM(page_views) AS sum_page_views,
		SUM(cart_adds) AS sum_cart_adds,
		SUM(purchase) AS sum_purchase,
		SUM(click) AS sum_click,
		SUM(impression) AS sum_impression
	FROM product_campaing_summary
	GROUP BY campaign_name, user_id
	)
	
	
/* Calculate average and total metrics for different campagns */
SELECT 
	campaign_name,
	COUNT(DISTINCT(user_id)) AS number_of_users,
	ROUND(AVG(sum_page_views)) AS avg_page_views,
	ROUND(AVG(sum_cart_adds)) AS avg_cart_adds,
	ROUND(AVG(sum_purchase)) AS avg_purchase,
	ROUND(AVG(sum_click)) AS avg_click,
	SUM(sum_page_views) AS total_page_views,
	SUM(sum_cart_adds) AS toal_cart_adds,
	SUM(sum_purchase) AS total_purchase,
	SUM(sum_click) AS total_click
FROM user_summary
WHERE sum_impression = 1  /* 1 - in case user had ad impression, 0 - otherwise
GROUP BY campaign_name
````

* Summary for users who had 'Ad Impression':

![image](https://user-images.githubusercontent.com/35038779/217804514-7ae915d6-5174-4161-8804-452774b935a9.png)

* Summary for users who had no 'Ad Impression':

![image](https://user-images.githubusercontent.com/35038779/217804582-a0909d73-7dba-4fe8-82af-e7f1ddf3edb5.png)


