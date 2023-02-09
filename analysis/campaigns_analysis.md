# Project: Clique-Bait

# Campaigns Analysis

## Contents

 1. [Generate visit summary table](#1-generate-visit-summary-table)
 2. [Campaign metrics analysis](#2-campaign-metrics-analysis)

---

### 1. Generate visit summary table

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

---

### 2. Campaign metrics analysis

- Identifying users who have received impressions during each campaign period and comparing each metric with other users who did not have an impression event


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
	ROUND(SUM(sum_page_views)/COUNT(DISTINCT(user_id)),1) AS page_views_per_user_rate,
	ROUND(SUM(sum_cart_adds)/COUNT(DISTINCT(user_id)),1) AS toal_cart_adds_per_user_rate,
	ROUND(SUM(sum_purchase)/COUNT(DISTINCT(user_id)),1) AS total_purchase_per_user_rate,
	ROUND(SUM(sum_click)/COUNT(DISTINCT(user_id)),1) AS total_click_per_user_rate
FROM user_summary
WHERE campaign_name IS NOT NULL AND sum_impression = 1 /* 1 - in case user had ad impression, 0 - otherwise */
GROUP BY campaign_name
````

* Summary for users who had 'Ad Impression':

![image](https://user-images.githubusercontent.com/35038779/217814312-b6edbfd5-a0fe-4572-96a9-d478549e3398.png)


* Summary for users who had no 'Ad Impression':

![image](https://user-images.githubusercontent.com/35038779/217814253-d7207a9e-5e64-43f8-8055-29ea2dfa9c5d.png)


As we can tell from two tables above for campangs with 'Ad Impression' and without it:
	1. Adding 'Ad Impression' increased every metric
	2. Purchase rate is two times more for users with 'Ad Impression'
	3. Campaign 'Half Off - Treat Your Shelf' - showed the best results among others 


