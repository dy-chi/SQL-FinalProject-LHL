What issues will you address by cleaning the data?





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
