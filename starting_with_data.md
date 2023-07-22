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
SELECT fullvisitorid, COUNT(DISTINCT productsku)
FROM public.all_sessions
GROUP BY fullvisitorid
ORDER BY count DESC
-- Only 0.06% of unqiue visitor sessions included a sale. Which seems very low. 
WITH cte_sales_by_vis_sesh AS(
			SELECT visitor_sessions_day_id, COUNT(transactionid) as sales_count
			FROM public.all_sessions
			GROUP BY visitor_sessions_day_id
			ORDER BY sales_count  DESC
	)
SELECT SUM(CASE WHEN sales_count <> 0 THEN 1 ELSE 0 END), COUNT(*), ((SUM(CASE WHEN sales_count <> 0 THEN 1 ELSE 0 END)::numeric/COUNT(*)::numeric)*100)::numeric (10,2)
FROM cte_sales_by_vis_sesh

Question 1: What is the average time a user spends on a page

SQL Queries:

Answer: 



Question 2: what type of page do users spend the most time on

SQL Queries:

Answer:



Question 3: what is the most common referral for users that stay on the site for 0 seconds, is signficant difference from the general trend of referrals? 

SQL Queries:
SELECT 
    channelgrouping, 
    SUM(CASE WHEN timeonsite = '00:00:00' THEN 1 ELSE 0 END) as cg_count_time0,
    (
        SUM(CASE WHEN timeonsite = '00:00:00' THEN 1 ELSE 0 END)
        / (SELECT COUNT(*) as total_rows FROM public.visitor_sessions_pk)::numeric(10,2) * 100
    )::numeric(10,2) as cg_count_time0_percent,		
    COUNT(timeonsite)  as cg_count,
    (
        COUNT(timeonsite)
        / (SELECT COUNT(*) as total_rows FROM public.visitor_sessions_pk)::numeric(10,2) * 100
    )::numeric(10,2) as cg_count_percent
FROM public.visitor_sessions_pk
GROUP BY channelgrouping
ORDER BY cg_count_time0 DESC;

Answer: There is a large increase in non-zero timeonsite when channelgrouping is a referral. FROM only  4.4% of sessions with refferal spent 0 time of site, this is a large decrease from  17.%  all traffic being a referral.





Question 4: 

SQL Queries:

Answer:



Question 5: 

SQL Queries:

Answer:
