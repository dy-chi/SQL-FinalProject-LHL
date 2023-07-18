What issues will you address by cleaning the data?

-- Investigate values where there are duplicates in visitid and if hey should be removed
WITH cte_visitid_dup AS(SELECT visitid, count(*)
FROM public.all_sessions
GROUP BY visitid
HAVING COUNT(visitid) > 1
ORDER BY visitid)

SELECT *
FROM cte_visitid_dup cte
LEFT JOIN public.all_sessions a_s USING(visitid)
ORDER BY visitid



-- duplicate values should not be removed, as they contain unqiue information, a true primary key must be found

---All of the duplicate visitid should share the same date, timeonsite, page views, etc. this query will see if there are any incorrect values  



WITH cte_visitid_dup AS(SELECT visitid, count(*)
FROM public.all_sessions
GROUP BY visitid
HAVING COUNT(visitid) > 1
ORDER BY visitid)

SELECT visitid
FROM cte_visitid_dup cte
LEFT JOIN public.all_sessions a_s USING(visitid)
GROUP BY visitid 
HAVING COUNT (distinct fullvisitorid) > 1

WITH cte_visitid_dup AS(SELECT visitid, count(*)
FROM public.all_sessions
GROUP BY visitid
HAVING COUNT(visitid) > 1
ORDER BY visitid)

SELECT visitid, count( distinct date) as distinct_count_date, min(fullvisitorid) min_visitor_id, max(fullvisitorid) as max_visitor_id, count(distinct fullvisitorid) as distinct_count_visitor_id, 
count(distinct timeonsite) as distinct_count_time_on_site, max(timeonsite) as max_timeonsite, max(date), min(date)
FROM cte_visitid_dup cte
LEFT JOIN public.all_sessions a_s USING(visitid)
GROUP BY visitid 
HAVING COUNT (distinct timeonsite) > 1

--The result shows there a 
--1) a handful of values with 2 fullvisitorids for one visitid, possibly switching accounts while on the same machine
--2)One visitid with multiple dates, the dates are consecutive  so it was likely the case of a user being on the site across midnight
-- If I create a primiary key that combines visit_id, visitor_id, and day (extracted from date), we will have useful primary key, because the analytics column also ---has these columns 



--Changing data types from TEXT

--ID colums to varchar with respective max lengths
--fullvisitor id is 16-19 *normally 19 digits and visitid is 10 digits long
--would have to confirm wether any fullvisitorid id with less than 19 digits is an error

ALTER TABLE public.all_sessions
ALTER COLUMN fullvisitorid TYPE varchar(19) USING fullvisitorid::varchar(19),
ALTER COLUMN visitid TYPE varchar(10) USING visitid::varchar(10);

ALTER TABLE public.analytics_distinct
ALTER COLUMN fullvisitorid TYPE varchar(19) USING fullvisitorid::varchar(19),
ALTER COLUMN visitid TYPE varchar(10) USING visitid::varchar(10);


--Alter datatype to date

ALTER TABLE public.all_sessions
ALTER COLUMN date TYPE date USING TO_DATE(date,'YYYYMMDD');

ALTER TABLE public.all_sessions;
ALTER COLUMN date TYPE date USING TO_DATE(date,'YYYYMMDD');



Queries:
Below, provide the SQL queries you used to clean your data.
Analytics table 


ISSUE 1 
--Duplicate Rows 

SELECT DISTINCT  *

FROM public.analytics;

SELECT   *

FROM public.analytics;

--1,739,308 distinct rows, total rows 4,301,112

--Delete duplicates by creating new table analytics_distinct

CREATE TABLE  analytics_distinct 
AS
SELECT DISTINCT *
FROM public.analytics

ISSUE 2: more than one distinct  fullvisitorid for each distinct visitorid.
----------------------------------------------------------------------
-- I assume there should only be 1 vistor id for each visitid per site therefore it is unexpected for 
-- duplicate visitor ids for what appears to be an identifier for a unique visit
SELECT DISTINCT visitid, COUNT( DISTINCT fullvisitorid) as duplicate_full_vis_id
FROM public.analytics
GROUP BY visitid
ORDER BY duplicate_full_vis_id DESC
-- The above query finds 1727 visitids with multiple fullvisitorids
-- should all entries associated with duplicate entries be deleted?
-- The following shows what the  fullvisitorid are like for each visit 

SELECT DISTINCT visitid, COUNT( DISTINCT fullvisitorid) as duplicate_full_vis_id, array_agg(distinct fullvisitorid)
FROM public.analytics
GROUP BY visitid
HAVING COUNT( DISTINCT fullvisitorid) > 1
ORDER BY duplicate_full_vis_id DESC

---Perhaps these are sessions on the website where a user cahnged accounts part way through the same visit? 
--------------------------------------------------------------------------

ISSUE 3: There are multiple rows with the same visitnumber and visitid

SELECT visitid,  array_agg(visitnumber) as visitnumber
FROM public.analytics
GROUP BY visitid

-- The visitor id ""1500643403" has many entries rows with the same visitnumber of '1' so I will investigate further to see if these rows have a unqiue feature or are they duplicates 

SELECT  *
FROM public.analytics
WHERE visitid = '1500643403'

Returns 465 rows

SELECT DISTINCT  *
FROM public.analytics
WHERE visitid = '1500643403'

Returns 47 rows
So there are duplicates, but visitid is not a primary key for this table.
Out of theese 47 rows the distinct feature is the unitprice 

PERHAPS A CONCATONATED KEY WITH visitid AND unitprice IS NEEDED....

SELECT DISTINCT  visitid, unit_price
FROM public.analytics


GROUP BY visitid, unit_price

returns 1,676,192 rows, notably close to the total distinct rows 1,739,308 


----------------------------------------------------------------
ISSUE 4: There are visitids that have a minimum visitnumber above 1 indicating this data may only be a snapshot of the full dataset


ISSUE  : There are some unit_price 0 values

-- create a table that has entries for each distinct visit with a distinct visitor id for each day of the visit, called visitor_session, all data that is distinct ---to this tables primary key: visitor_session_id DISTINCT CONCAT(visitid,'_',  fullvisitorid,'_', EXTRACT(DAY from date)) will live here. 

CREATE TABLE visitor_session (
    visitor_session_day VARCHAR(31) PRIMARY KEY,
    visitid VARCHAR(10) NOT NULL,
    fullvisitorid VARCHAR(19) NOT NULL,
	country TEXT,
	city TEXT,
	pageviews smallint,
	sessionqualitydim TEXT,
	date DATE NOT NULL,
	visitstarttime TIMESTAMP,
	timeonsite INTERVAL
	
);


-- create sessions table 

CREATE TABLE visitor_session2 AS   
	SELECT
		visitid,
		fullvisitorid,
		channelgrouping,
		pageviews,
		date,
		timeonsite::INTERVAL,
		'all_sessions' as source
	FROM public.all_sessions

UNION 
	SELECT
		visitid,
		fullvisitorid,
		channelgrouping,
		pageviews,
		date,
		timeonsite::INTERVAL,
		'analytics_distinct' as source
	FROM public.analytics_distinct


--create primary key column
ALTER TABLE public.visitor_session2
ADD COLUMN visitor_sessions_day_id VARCHAR(34)
--populate primary key column

ALTER TABLE public.all_sessions
ADD COLUMN visitor_sessions_day_id VARCHAR(34)

