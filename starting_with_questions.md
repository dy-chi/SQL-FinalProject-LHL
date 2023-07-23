Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**

--Considering the column totaltransactionrevenue is ambigious, we do not know over which data the totals are summing, I've opted to answer the question using only data where we have transactionids. Below I sum the transaction revenue with transactionids and use the country and city combination given at the row of the transaction, as opposed to using the latest country, city combination found in the visitor_info  table.


Answer cities:

"United States"  "Sunnyvale"	849.24
"Australia"	"Sydney"	358.00
"United States"	"Austin"	35.78
"United States"	"Mountain View"	8.98

Answer Countries 

"United States"	3208.95	
"Australia"	358.00	


SELECT country, city, 
	(SUM(tr.transactionrevenue)/1000000) ::numeric(10,2) as sum_transactions,
	COUNT( (city, country)) as num_transactions 
FROM public.transactions tr
JOIN public.all_sessions USING(transactionid)
GROUP BY country, city
HAVING count(city) >= 1
ORDER BY sum_transactions DESC

SELECT country, 
	(SUM(tr.transactionrevenue)/1000000) ::numeric(10,2) as sum_transactions,
	COUNT((country)) as num_transactions 
FROM public.transactions tr
JOIN public.all_sessions USING(transactionid)
GROUP BY country
HAVING count(country) >= 1

**Question 2: What is the average number of products ordered from visitors in each city and country?**

--Things to note: for all_sessions, not all sales have a recorded country or city, this may bias the data if only certain countries are not recorded 
--For analytics table, no city or country columns are recorded, so country is assumed to be the same as that in the visitor_info column which only has data from all_sessions, so only visitors with data in both all_sessions and analytics will have country or city data and it may not be up to date.

SELECT vi.country, AVG(total_products_sold) ::numeric(10,2) as avg_product_sold, count(total_products_sold) total_sales_by_country
FROM public.visitor_session_with_source vsws
JOIN public.visitor_info6 vi USING(fullvisitorid)
WHERE total_products_sold is not NULL
	AND total_products_sold <> 0
	AND vi.country is not NULL
GROUP BY vi.country
ORDER BY avg_product_sold DESC
-- Spain has the highest average products sold, but 3.82 from the US is the only meaningful average because the sample size is large enough
COUNTRY		AVG_PROD COUNT
"Spain"		10.00	1
"United States"	3.82	89
"Colombia"	1.00	1
"Finland"	1.00	1
"France"	1.00	2
"Argentina"	1.00	1
"Ireland"	1.00	2
"Mexico"	1.00	1
"Switzerland"	1.00	2
"India"	1.00	1
"Canada"	1.00	3

For cities, Sunvale has the highest average 
COUNTRY		CITY		AVG_PROD COUNT
"United States"	"Sunnyvale"	10.43	7
"Spain"		"Madrid"	10.00	1
"United States"	"Salem"	8.00	1
"United States"	"Seattle"	4.00	2
"United States"	"Atlanta"	4.00	1
"United States"	"San Francisco"	2.40	10
"United States"	"Houston"	2.00	2
"United States"	"Chicago"	1.67	3
"United States"	"New York"	1.36	11
"United States"	"Mountain View"	1.35	17
"United States"	"San Jose"	1.00	1
"United States"	"Detroit"	1.00	2
"Ireland"	"Dublin"	1.00	2
"Switzerland"	"Zurich"	1.00	2
"United States"	"(not set)"	1.00	2
"United States"	"Ann Arbor"	1.00	2
"United States"	"Columbus"	1.00	1
"United States"	"Dallas"	1.00	2
"India"		"Bengaluru"	1.00	1
"United States"	"Los Angeles"	1.00	1
"United States"	"Palo Alto"	1.00	3

**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**

-- not enought data to say, the data from all sessions is mostly showing data of people navigating the web page rather than orders. There are only 9 orders made with high quality data (transactionid etc), not enough to show patterns in my view, perhaps with further investigationing I might be able to link productskus to sales in analytics  

SQL Queries:

SELECT  v2productname, alls.productsku, alls.country, alls.city
FROM public.all_sessions alls
JOIN public.transactions USING(transactionid)


Answer:

"Nest® Cam Indoor Security Camera - USA"	"GGOENEBB078899"	"United States"	"Sunnyvale"
"Nest® Cam Indoor Security Camera - USA"	"GGOENEBB078899"	"Australia"	"Sydney"
"Google Men's 100% Cotton Short Sleeve Hero Tee Black"	"GGOEGAAB010516"	"United States"	"Austin"
"Waze Mobile Phone Vent Mount"	"GGOEWEBB082699"	"United States"	"Mountain View"
"SPF-15 Slim & Slender Lip Balm"	"GGOEGCBQ016499"	"United States"	"Sunnyvale"
"Nest® Cam Indoor Security Camera - USA"	"GGOENEBB078899"	"United States"	
"Compact Bluetooth Speaker"	"GGOEGEVJ023999"	"United States"	
"Reusable Shopping Bag"	"GGOEGBJR018199"	"United States"	
"Google Bongo Cupholder Bluetooth Speaker"	"GGOEGEVB070899"	"United States"	





**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:


Answer:

--There are many products without a sku in the analytics csv. So I'm not sure how, or if  they are meant to be connected. When looking at only transactions 
-- in the all_sessions csv there are only 9 values with transaction ids so I would say the data returned for top selling product is pretty useless. But it looks to --be resuable bags, cheap products with many units per order. Or the "Nest® Cam Indoor Security Camera - USA" if we are talking about sale price and number of 
--distinct transactions 

SELECT  country, productsku, array_agg(v2productname), SUM(productquantity::integer)
FROM public.all_sessions alls
JOIN public.transactions USING(transactionid)

GROUP BY country, productsku

ORDER BY SUM DESC





**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:
SELECT vi.country, SUM(sum_transactions) ::numeric(10,2) as sum_sales_by_country
FROM public.visitor_sessions_pk vspk
JOIN public.visitor_info vi USING(fullvisitorid)
WHERE  sum_transactions is not NULL
	AND sum_transactions <> 0
	AND vi.country is not NULL
GROUP BY vi.country
ORDER BY sum_sales_by_country DESC


Answer: The vast majoirty of revenue is from the USA with minor ammounts from other countries. However there is lots of missing country data from the analytics column

"United States"	58566.35
"Canada"	447.48
"Australia"	358.00
"Germany"	69.98
"Japan"	30.88
"Switzerland"	16.99









