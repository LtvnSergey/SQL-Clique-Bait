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
SELECT campaign_name,impression,
	   ROUND(SUM(page_views)/COUNT(DISTINCT(visit_id)), 2) AS page_views_per_visit,
	   ROUND(SUM(cart_adds)/COUNT(DISTINCT(visit_id)), 2) AS cart_add_per_visit,
	   ROUND(SUM(purchase)/COUNT(DISTINCT(visit_id)), 2) AS purchase_per_visit,
	   ROUND(SUM(click)/COUNT(DISTINCT(visit_id)), 2) AS click_per_visit

FROM product_campaing_summary
WHERE campaign_name IS NOT NULL
GROUP BY campaign_name, impression
````
![image](https://user-images.githubusercontent.com/35038779/217835564-acaa3a50-185a-4a56-8749-aab49dd8af74.png)


- As we can tell from table above for campangs with 'Ad Impression' and without it:
	1. Adding 'Ad Impression' increased every metric
	2. Purchase rate is two times more for users with 'Ad Impression'

--

- Lets check whereever clicking on an impression lead to higher purchase rates

````sql
SELECT campaign_name,
	   click,
	   ROUND(SUM(purchase)/COUNT(DISTINCT(visit_id)), 2) AS purchase_per_visit
FROM product_campaing_summary
WHERE impression=1 AND campaign_name IS NOT NULL
GROUP BY campaign_name, click
````

![image](https://user-images.githubusercontent.com/35038779/217837565-89920598-32d7-4bdc-9f5a-671c81c13f8c.png)

- According to the table above - clicking on an impression lead to higher purchase rates

- More significantly this effect is pronounced for campaigns 'BOGOF - Fishing For Compliments' and 'Half Off - Treat Your Shellf(ish)'
