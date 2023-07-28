Consider the data you have available to you. You can use the data to: - find all duplicate records - find the total number of unique visitors (fullVisitorID) - find the total number of unique visitors by referring sites - find each unique product viewed by each visitor - compute the percentage of visitors to the site that actually makes a purchase


-- There are 130345 unique visitors
SELECT channelgrouping, COUNT(DISTINCT fullvisitorid)
FROM public.visitor_sessions_pk
GROUP BY channelgrouping

-- Most pathways are from organic search, then direct, then referral 
SELECT channelgrouping, COUNT(DISTINCT fullvisitorid) as unique_visits_by_site
FROM public.visitor_sessions_pk
GROUP BY channelgrouping
ORDER BY unique_visits_by_site DESC

--The visitor who viewed the most products was, with 10

	WITH cte_visitor AS(
	SELECT fullvisitorid
	FROM public.visitor_sessions_pk
	GROUP BY fullvisitorid)

SELECT 
	fullvisitorid,
	COUNT(DISTINCT productsku ) as num_distinct_prods
FROM
	public.all_sessions alls

JOIN 
	cte_visitor
	USING(fullvisitorid)
GROUP BY 
	fullvisitorid
ORDER BY 
	num_distinct_prods DESC
	


-- Only 3.96% of  visitor sessions included a sale. Which seems low but perhaps normal for this website
WITH cte_sales AS (
    SELECT
        visitor_sessions_day_id,
        SUM(total_num_transactions) AS sales_count
    FROM
        public.visitor_sessions_pk
    GROUP BY
        visitor_sessions_day_id
)
SELECT
    SUM(CASE WHEN sales_count IS NOT NULL THEN 1 ELSE 0 END) AS sessions_wsales_count,
    COUNT(*) AS total_sessions_count,
    ((100*SUM(CASE WHEN sales_count IS NOT NULL THEN 1 ELSE 0 END)::numeric) / COUNT(*)::numeric)::numeric(10,2) AS purchase_per_vis_percent
FROM
    cte_sales;

Question 1: What is the average time a user spends on a certain type of page

SQL Queries:
SELECT pagepathlevel1, AVG(time_on_page) 

FROM public.all_sessions
GROUP BY pagepathlevel1
ORDER BY avg DESC

Answer: Most time is spend on the review order, payment and basket pages. 

"/revieworder.html"	"00:08:05.2"
"/payment.html"	"00:07:55.769231"
"/basket.html"	"00:06:58"



Question 2: what type of page do users spend the most time on
-- Common Table Expression (CTE) to get unique visitor sessions based on fullvisitorid and visitid
SQL Queries:
	WITH cte_visitor AS(
	SELECT fullvisitorid,visitid
	FROM public.visitor_sessions_pk
	GROUP BY fullvisitorid,visitid)

-- Main Query: Calculate the cumulative time spent on each page (pagetitle) by visitors from analytics and all_sessions sources
SELECT 

	pagetitle,
	SUM(time_on_page) as time_on_page, -- Cumulative time on page over the same visitorid
	pagepathlevel1
FROM
	public.all_sessions alls

JOIN 
	cte_visitor cte
	USING(fullvisitorid)
WHERE LENGTH(pagetitle) < 200 -- Filter out results with pagetitles longer than 200 characters, as they are questionable and can harm readability

GROUP BY 
	pagetitle,pagepathlevel1
	
ORDER BY time_on_page DESC

Answer: Shop by brand from the youtube or google path to the Google Merchandise Store is the webpage users spend the most time on, not on average, but as a total amount of time spent on the website. This is different from the answer in answer 1 because there are likley lots of short visits to these pages with short times on page

"YouTube | Shop by Brand | Google Merchandise Store"	"31:50:22"	"/google+redesign/"
"Google | Shop by Brand | Google Merchandise Store"	"29:52:14"	"/google+redesign/"
"Men's T-Shirts | Apparel | Google Merchandise Store"	"28:54:58"	"/google+redesign/"
"Store search results"	"23:57:19"	"/asearch.html"
"Men's Outerwear | Apparel | Google Merchandise Store"	"22:51:01"	"/google+redesign/"
"Accessories | Google Merchandise Store"	"22:37:36"	"/google+redesign/"
"Electronics | Google Merchandise Store"	"21:07:29"	"/google+redesign/"
"Apparel | Google Merchandise Store"	"18:39:15"	"/google+redesign/"


Question 3: what is the most common referral for users that stay on the site for 0 seconds, is signficant difference from the general trend of referrals? 

SQL Queries:
-- Main Query: Calculate statistics related to time spent on each channel grouping

SELECT
    channelgrouping, -- The name of the channel grouping
    SUM(CASE WHEN timeonsite = '00:00:00' THEN 1 ELSE 0 END) AS cg_count_time0, -- Count of sessions with time spent = 0
    ((SUM(CASE WHEN timeonsite = '00:00:00' THEN 1 ELSE 0 END)::NUMERIC(10, 2) /COUNT(timeonsite)::NUMERIC(10, 2)) * 100)::NUMERIC(10, 2) AS cg_count_time0_percent, -- Percentage of sessions with time spent = 0

    COUNT(timeonsite) AS cg_count, -- Total count of sessions in the channel grouping
    (COUNT(timeonsite) / (SELECT COUNT(*) AS total_rows FROM public.visitor_sessions_pk)::NUMERIC(10, 2) * 100)::NUMERIC(10, 2) AS cg_count_percent -- Percentage of total sessions in the channel grouping
FROM public.visitor_sessions_pk

-- Group the results by channel grouping
GROUP BY channelgrouping

-- Order the results by the count of sessions with time spent = 0 in descending order
ORDER BY cg_count_time0 DESC;

Answer: There is a large increase in non-zero timeonsite when channelgrouping is a referral. FROM only  7.4% of sessions with referral spent 0 time of site, whereas 59%% of visits with social referral spent 0 time on the site.  

Channelgrouping time_0_count    total channel grouping count  
"Organic Search"32967	39.90	82634	51.16
"Direct"	8702	30.95	28116	17.41
"Social"	7080	59.95	11810	7.31
"Referral"	2276	7.42	30683	19.00
"Paid Search"	474	9.33	5081	3.15




Question 4: How much has the overall time spent on site changed over time?

SQL Queries:
SELECT 
	TO_CHAR(date, 'YYMM') AS YYMM_yearmonth, 
	SUM(timeonsite) as sum_timeonsite,
	(EXTRACT(hours FROM SUM(timeonsite))/24)::numeric(10,2) as sum_timeonsite_days

FROM
	public.visitor_sessions_pk
GROUP BY TO_CHAR(date, 'YYMM')

FROM
	public.visitor_sessions_pk
GROUP BY TO_CHAR(date, 'YYMM')
Answer: There is a huge increase in the overall time spent on site in MAY-JULY 2017, from a background of 54 hours to 
This might be due to an error or more likey the extent of the data in the csvs we were given, this was found to be the case

SELECT MAX(date), MIN(date)
FROM public.analytics_distinct



Question 5: Is there any improvment in sales over the weekend?

SQL Queries:
CREATE OR REPLACE FUNCTION is_weekend(date_val date)
RETURNS boolean AS
$$
BEGIN
    RETURN EXTRACT(ISODOW FROM date_val) IN (6, 7); -- 6 is Saturday, 7 is Sunday in ISO week numbering
END;
$$
LANGUAGE plpgsql;

-- Main Query: Calculate the average of sum_transactions for weekends and weekdays

SELECT 
    is_weekend(date) AS is_weekend, -- Check if the date is a weekend (Saturday or Sunday)
    AVG(sum_transactions)::numeric(10,2) -- Calculate the average of sum_transactions and round to 2 decimal places
FROM 
    public.visitor_sessions_pk
WHERE 
    date BETWEEN '2017-05-01' AND '2017-08-01' -- Filter data for dates between May 1, 2017, and August 1, 2017
GROUP BY 
    is_weekend(date); -- Group the results by is_weekend to separate weekends and weekdays


Answer: There is a modest increase in sales revenue during the weekdays compared to the weekend  
is_weekend, 		Avg sum_transaction_by_session 
false			174.81
true			153.66
