# Project: Clique-Bait

# Campaigns Analysis

## Questions



WITH 
	users_visits AS (
		SELECT 
			u.user_id,
			e.visit_id,
			MIN(event_time) OVER(PARTITION BY visit_id) AS visit_start_time,
			e.event_type
		FROM clique_bait.events AS e
		LEFT JOIN
		clique_bait.users AS u
		ON e.cookie_id=u.cookie_id
	),
		
	page_views AS (
		SELECT visit_id, COUNT(*) AS views
		FROM users_visits
		WHERE event_type IN 
			(SELECT event_type FROM clique_bait.event_identifier 
			 WHERE event_name='Page View')
		GROUP BY visit_id
	),
	
	cart_adds AS (
		SELECT visit_id, COUNT(*) AS cart_adds
		FROM users_visits
		WHERE event_type IN 
			(SELECT event_type FROM clique_bait.event_identifier 
			 WHERE event_name='Add to Cart')
		GROUP BY visit_id
	),
	
	purchase AS (
		SELECT DISTINCT visit_id, 
			CASE WHEN visit_id IN
				(SELECT DISTINCT visit_id
				FROM
				(SELECT visit_id,
					CASE WHEN event_type IN 
						(SELECT event_type FROM clique_bait.event_identifier 
						 WHERE event_name='Purchase') 
						THEN 1 ELSE 0 END purchase
				FROM users_visits) AS p
				WHERE purchase = 1)
				THEN 1 ELSE 0 END purchase
		FROM users_visits
	)
	
	campagn_name AS (
		SELECT DISTINCT visit_id
			campaing_name
		FROM 
	)
	
SELECT * 
FROM purchase
