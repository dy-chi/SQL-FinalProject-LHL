-- Cleaning Data in PostgreSQL (psql)

--  Goal 1: Identify possible entities, dimensions, facts and relationships (if any) between tables

-- List of Entities  unique identifiers, facts, found after investigating the tables 
-- fullvisitorid -> visitor 

--visitid -> likely to be a page visit, time_on_page 
--visitor_sessions_day_id visitor page visit -> (combination of fullvisitorid, visitid and day of month (since there seem to be duplicate rows when crossing midnight on the same visit))
        - channel referral
        - country, city 
--transactions -> transactionid 
--pageid -> not not created 
--channel referral
--products
        - productname
        - product price 
        - etc

-- Step 1: Investigate values where there are duplicates in visitid to decide whether to remove them.

-- Find visitids with duplicate counts > 1.
WITH cte_visitid_dup AS (
    SELECT visitid, COUNT(*) AS duplicate_count
    FROM public.all_sessions
    GROUP BY visitid
    HAVING COUNT(visitid) > 1
    ORDER BY visitid
)

-- Show rows with duplicate visitids and their corresponding data.
SELECT *
FROM cte_visitid_dup cte
LEFT JOIN public.all_sessions a_s USING(visitid)
ORDER BY visitid;

-- Step 2: Identify duplicate visitids with different fullvisitorids and investigate further.

-- Find visitids with multiple distinct fullvisitorids.
WITH cte_visitid_dup AS (
    SELECT visitid, COUNT(DISTINCT fullvisitorid) AS duplicate_full_vis_id
    FROM public.all_sessions
    GROUP BY visitid
    HAVING COUNT(DISTINCT fullvisitorid) > 1
    ORDER BY duplicate_full_vis_id DESC
)

-- Show visitids with multiple distinct fullvisitorids and their corresponding fullvisitorids.
SELECT visitid, COUNT(DISTINCT fullvisitorid) AS duplicate_full_vis_id, array_agg(DISTINCT fullvisitorid)
FROM public.all_sessions
GROUP BY visitid
HAVING COUNT(DISTINCT fullvisitorid) > 1
ORDER BY duplicate_full_vis_id DESC;

-- Step 4: Investigate duplicate rows with the same visitnumber and visitid.
-- Show visitids with multiple visitnumbers.

SELECT visitid, array_agg(visitnumber) AS visitnumber
FROM public.all_sessions
GROUP BY visitid;

-- Step 5: Clean the data by removing duplicates and adding appropriate primary key.

-- Create a new table 'analytics_distinct' with distinct rows to remove duplicates.
CREATE TABLE analytics_distinct AS
SELECT DISTINCT *
FROM public.analytics;
----------------------------------------------------------------------------------------
---GOAL 2: Alter datatypes from the original text datatype to more fitting datatypes Change 
-- Step 1: Change ID columns to VARCHAR with respective max lengths.
-- 'fullvisitorid' is 16-19 characters (normally 19 digits), and 'visitid' is 10 digits long.
-- Make sure there are no errors with 'fullvisitorid' values less than 19 digits.
ALTER TABLE public.all_sessions
ALTER COLUMN fullvisitorid TYPE VARCHAR(19) USING fullvisitorid::VARCHAR(19),
ALTER COLUMN visitid TYPE VARCHAR(10) USING visitid::VARCHAR(10);

ALTER TABLE public.analytics_distinct
ALTER COLUMN fullvisitorid TYPE VARCHAR(19) USING fullvisitorid::VARCHAR(19),
ALTER COLUMN visitid TYPE VARCHAR(10) USING visitid::VARCHAR(10);
-- Step 1:
-- Alter the data type to date.
ALTER TABLE public.all_sessions
ALTER COLUMN date TYPE DATE USING TO_DATE(date, 'YYYYMMDD');
-- Step 2: Change date column from TEXT to DATE 
ALTER TABLE public.all_sessions
ALTER COLUMN date TYPE DATE USING TO_DATE(date, 'YYYYMMDD');

-----------------------------------------
---GOAL3 Create new tables were there are no duplicates in the id column from all_sessions and analytics sources with any relevant facts concerning the visitor_session_day_id entity
-- Step 1: Create new tables 'visitor_session' with distinct data.

-- Create 'visitor_session' table with unique rows based on the primary key: 'visitor_session_day_id.'
CREATE TABLE visitor_session (
    visitor_session_day_id VARCHAR(31) PRIMARY KEY,
    visitid VARCHAR(10) NOT NULL,
    fullvisitorid VARCHAR(19) NOT NULL,
    country TEXT,
    city TEXT,
    pageviews SMALLINT,
    sessionqualitydim TEXT,
    date DATE NOT NULL,
    visitstarttime TIMESTAMP,
    timeonsite INTERVAL
);

-- Create 'visitor_session_source' table with distinct rows from 'all_sessions' and 'analytics_distinct' tables.
CREATE TABLE visitor_session_source AS   
SELECT
    visitid,
    fullvisitorid,
    channelgrouping,
    pageviews,
    date,
    timeonsite::INTERVAL,
    'all_sessions' AS source
FROM public.all_sessions

UNION 

SELECT
    visitid,
    fullvisitorid,
    channelgrouping,
    pageviews,
    date,
    timeonsite::INTERVAL,
    'analytics_distinct' AS source
FROM public.analytics_distinct;

-- Step 2: Add primary key column to 'visitor_session_source' table and populate it.

-- Add the 'visitor_sessions_day_id' column to 'visitor_session_source'.
ALTER TABLE public.visitor_session_visitor_session_source
ADD COLUMN visitor_sessions_day_id VARCHAR(34);

-- Populate the 'visitor_sessions_day_id' column with unique values.
UPDATE public.visitor_sessionvisitor_session_source
SET visitor_sessions_day_id = CONCAT(visitid, '_', fullvisitorid, '_', EXTRACT(DAY FROM date));

-- Step 2: Remove duplicate entries from 'visitor_session_source' based on the primary key.

-- Remove duplicate entries while keeping the row with the minimum 'visitor_sessions_day_id'.
DELETE FROM
    public.visitor_session_source a
USING public.visitor_session_source b
WHERE
    a.visitor_sessions_day_id < b.visitor_sessions_day_id
    AND a.visitid = b.visitid
    AND a.date = b.date
    AND a.fullvisitorid = b.fullvisitorid
    AND a.pageviews = b.pageviews
    AND a.timeonsite = b.timeonsite;
-- Step 3:  Primary Key Checks
-- Check if primary key has nulls
SELECT *
FROM public.visitor_sessions_pk
WHERE visitor_sessions_day_id is NULL;
-- Check if primary key has duplicates,  values returned must be equal
SELECT COUNT(visitor_sessions_day_id), COUNT(DISTINCT visitor_sessions_day_id)
FROM public.visitor_sessions_pk;

-- Checks if primary key has incorrect values
SELECT *
FROM public.visitor_sessions_pk
WHERE visitor_sessions_day_id ~ '[^0-9_]' OR LENGTH(visitor_sessions_day_id) < 28;

-- Step 4  Check each column in table for nulls
SELECT COUNT(visitid) as visitid_count, COUNT(fullvisitorid) as fullvisid_count, COUNT(channelgrouping) as channelgrouping_count, COUNT(pageviews) as pageview_count, COUNT(date) as date_count, COUNT(timeonsite) as timeonsite_count, COUNT(country) as country_count, COUNT(city) as city_count, COUNT(total_num_transactions) as t_count, COUNT(sum_transactions) as sum_count
FROM
public.visitor_sessions_pk

-- pageview column has (161512 - 161481) has 31 NULLS associated with EVENT type 
-- Replace NULLS with zero as this provides more information
UPDATE public.visitor_sessions_pk
SET pageviews = COALESCE(pageviews, '0');
-- I will also change the datatype to smallint
ALTER TABLE public.visitor_sessions_pk
ALTER COLUMN pageviews TYPE smallint USING pageviews::smallint;

---timeonesite column has (161512 - 109607) 51,905 nulls 
-- Replace NULLS with zero as this provides more information, perhaps this means a tab is opened but the user did not interact, or someother action that might be worth recording
-- country column is mostly NULLS having only 14539 values out of 161512 
-- Looking in the all_sessions table there are no transactions where the country column is blank, so I believe it is acceptable to leave this column with nulls
--city column has the same number of nulls as country and the nulls can remain for the same reasons
SELECT *
FROM public.visitor_sessions_pk
WHERE country is NULL and city is not null 
	OR  city is NULL and country is not null;
----------------------------------------------------------------------------------------------
--GOAL 4 Create a table with a transactions entity holding all unique transactions
-- into the table will be Into the transactions table will be transactionid, transactionrevenue, visitor_sessions_day_id, date, currencycode
--First each of these columns must be cleaned  in the  all_sessions table

--when on the order completed page, total revenue should be equal to transaction revenue if the number of transactions is 1, 
--this is consistent with other orders having values in the transaction revenue column with 1 transaction

UPDATE public.all_sessions 
SET transactionrevenue = COALESCE(transactionrevenue, totaltransactionrevenue::text)
	WHERE pagepathlevel1 = '/ordercompleted.html' 
		AND transactions = '1'
		AND transactionid is not NULL;
--Create Table Transactions
CREATE TABLE transactions

(transactionid VARCHAR(16),
transactionrevenue numeric,
visitor_sessions_day_id VARCHAR(34),
date date,
currencycode VARCHAR(5));

INSERT INTO transactions 
	(
	transactionid,
	transactionrevenue,
	visitor_sessions_day_id,
	date,
	currencycode)
SELECT 
	transactionid,
	transactionrevenue::numeric,
	visitor_sessions_day_id,
	date,
	currencycode
FROM public.all_sessions
WHERE transactionid IS NOT NULL;

---------------
--GOAL 5 Create a table for the information on visitors 
--Step 1 Look at relevant  columns to investigate 
SELECT COUNT(DISTINCT date), array_agg(country),  COUNT(DISTINCT country) as d_count_country, array_agg(city), COUNT(DISTINCT city) as dcount_city
FROM public.all_sessions

GROUP BY fullvisitorid
ORDER BY COUNT(DISTINCT country) DESC;

-- Step 2 Cleaning steps country, city
---Coalesce empty values NULL
---take most recent location for the visitor table
UPDATE public.all_sessions
SET city = COALESCE(
				NULLIF(city, 'not available in demo dataset'),
				NULLIF(city, '(not set)')
				)
WHERE city = 'not available in demo dataset' OR city = '(not set)' ;

--Step 3 

--Create Table visitor info 

CREATE TABLE visitor_info AS
	WITH cte_max_date 
		AS(SELECT 
			fullvisitorid, 
			max(date) as latest_date,
		   	max(visitid::integer) as latest_visitid,
			COUNT(DISTINCT visitid) as visit_count,
			SUM(time_on_page) as total_time_on_pages
		FROM public.all_sessions
		GROUP BY  fullvisitorid)

	SELECT DISTINCT
			alls.fullvisitorid,
			alls.country,
			alls.city,alls.totaltransactionrevenue,
			alls.timeonsite,
			alls.date,
			total_time_on_pages,
			visit_count
		FROM public.all_sessions alls
		JOIN cte_max_date cte ON alls.fullvisitorid = cte.fullvisitorid 
								AND alls.date = cte.latest_date
								AND alls.visitid::integer = cte.latest_visitid

ALTER TABLE public.visitor_info
ADD CONSTRAINT pk_constraint_fullvistorid PRIMARY KEY (fullvisitorid);

-----------------
-- GOAL 6 Populate public.visitor_session_with_source and public.visitor_session_pk  table with transactions data
--Step 1
WITH cte_analytics_distinct 
	AS(

	SELECT  
		MAX(visitid) as visitid, 
		MAX(fullvisitorid) as fullvisitorid,
		MAX(channelgrouping) as channelgrouping,
		MAX(pageviews) as pageviews,
		MAX(date) as date,
		MAX(timeonsite) as timeonsite,
		'analytics_distinct' as source,
		visitor_sessions_day_id,
		'replace' as country,
		'replace' as city,
		COUNT(revenue) as total_num_transactions,
		SUM(((revenue::numeric)/1000000)::numeric(10,2)) as sum_transactions 
	FROM public.analytics_distinct		
	WHERE revenue is not null
	GROUP BY visitor_sessions_day_id
	)
UPDATE public.visitor_session_with_source vs
	SET sum_transactions = cte.sum_transactions,
	total_num_transactions =cte.total_num_transactions
	
	FROM cte_analytics_distinct cte
	WHERE 	vs.visitor_sessions_day_id = cte.visitor_sessions_day_id
			AND vs.source = cte.source;
-----------------------------------------
CREATE TEMPORARY TABLE temp_table2 AS
WITH cte_analytics_distinct AS (
    SELECT  
        MAX(visitid) as visitid, 
        MAX(fullvisitorid) as fullvisitorid,
        MAX(channelgrouping) as channelgrouping,
        MAX(pageviews) as pageviews,
        MAX(date) as date,
        MAX(timeonsite) as timeonsite,
        'analytics_distinct' as source,
        visitor_sessions_day_id,
        'replace' as country,
        'replace' as city,
        COUNT(revenue) as total_num_transactions,
        SUM(((revenue::numeric)/1000000)::numeric(10,2)) as sum_transactions 
    FROM public.analytics_distinct		
    WHERE revenue is not null
    GROUP BY visitor_sessions_day_id
),
cte_groupby AS (
    SELECT visitor_sessions_day_id, 
        SUM(sum_transactions) as sum_transactions, 
        SUM(total_num_transactions) as total_num_transactions
    FROM cte_analytics_distinct
    GROUP BY visitor_sessions_day_id
			)

	SELECT visitor_sessions_day_id, sum_transactions, total_num_transactions
	FROM cte_groupby
	UNION 
	SELECT visitor_sessions_day_id, transactionrevenue/1000000 as sum_transactions, 1 as total_num_transactions
	FROM public.transactions
	ORDER BY sum_transactions DESC

UPDATE public.visitor_sessions_pk vspk
SET sum_transactions = tt.sum_transactions,
    total_num_transactions = tt.total_num_transactions
FROM temp_table2 tt
WHERE vspk.visitor_sessions_day_id = tt.visitor_sessions_day_id;

--Step 3 add total products sold column 

ALTER TABLE public.visitor_session_with_source
ADD COLUMN total_products_sold INTEGER;

WITH cte_analytics_distinct
	AS(

	SELECT  
		MAX(visitid) as visitid, 
		MAX(fullvisitorid) as fullvisitorid,
		MAX(channelgrouping) as channelgrouping,
		MAX(pageviews) as pageviews,
		MAX(date) as date,
		MAX(timeonsite) as timeonsite,
		'analytics_distinct' as source,
		visitor_sessions_day_id,
		'replace' as country,
		'replace' as city,
		SUM(units_sold::integer)::integer as units_sold,
		COUNT(revenue) as total_num_transactions,
		SUM(((revenue::numeric)/1000000)::numeric(10,2)) as sum_transactions 
	FROM public.analytics_distinct		
	WHERE revenue is not null
	GROUP BY visitor_sessions_day_id
	)
UPDATE public.visitor_session_with_source AS vsws
SET total_products_sold = (
  SELECT COALESCE(SUM(ad.units_sold), 0) + COALESCE(SUM(alls.productquantity::integer  ), 0)
  FROM cte_analytics_distinct AS ad
  FULL OUTER JOIN all_sessions AS alls ON ad.visitor_sessions_day_id = alls.visitor_sessions_day_id
  WHERE  alls.transactionid IS NOT NULL AND 
		vsws.visitor_sessions_day_id = ad.visitor_sessions_day_id 
		OR vsws.visitor_sessions_day_id = alls.visitor_sessions_day_id)



