# Project: Clique-Bait

# Campaigns Analysis

## Questions



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

