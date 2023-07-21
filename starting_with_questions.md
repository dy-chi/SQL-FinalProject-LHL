Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**

--Considering the column totoaltransactionrevenue is ambigious, we do not know over which data the totals are summing, I've opted to answer the question using only data where we have transactionids. Below I sum the transaction revenue with transactionids and use the country and city combination given at the row of the transaction, as opposed to using the latest country, city combination found in the visitor_info  table.


Answer cities:

"United States"  	"Sunnyvale"	849.24
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

--We don't have enough data to answer this properly, as the number of orders with known product quantities is low

SQL Queries:

SELECT country, AVG(productquantity::integer)
FROM public.transactions tr
JOIN public.all_sessions alls USING(transactionid)
WHERE country is not null 
GROUP BY country 

SELECT city, AVG(productquantity::integer)
FROM public.transactions tr
JOIN public.all_sessions alls USING(transactionid)
WHERE country is not null 
GROUP BY city 
Answer:
"Austin"	
"Mountain View"	
"Sunnyvale"	1.00000000000000000000
"Sydney"	

SELECT city, AVG(productquantity::integer)
FROM public.transactions tr
JOIN public.all_sessions alls USING(transactionid)
WHERE country is not null 
GROUP BY country 




**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**

-- not enought data to say, the data from all sessions is mostly showing data of people navigating a web page rather than orders. There are only 9 orders made, not enough to show patterns, 

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





**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:



Answer:







