What are your risk areas? Identify and describe them.

QA Process:
Describe your QA process and include the SQL queries used to execute it.

-- There is an assumption that the productsku fact in the all_sessions table is the only product included in the transaction
-- To check this I would see if the transaction revenue is about equal to the  productquantity*product price

SELECT productsku,
(transactionrevenue::numeric)/1000000 as transaction_rev,
(productprice::numeric)/1000000 as product_price,
(productquantity::integer) as product_quantity,
((productquantity::integer)*(productprice::numeric)/1000000)::numeric(10,2)
FROM public.all_sessions
WHERE transactionid is not NULL

-- It seems that many values do not add up and therefore my suspicious are correct that we will need more information on which products are actually being sold 
--before confidently aswering questions requiring product skus

---For the analytics table, I'm wondering if the same is true, does each row with revenue represent a purchase of a single sku or multiple.
-- In lieu of having productsku in this table I will assume the unit_price is somewhat rarely shared by multiple products. So if there were ever  more unique prices per uservisit than revenue entries that would mean that each row does not represent the sale of one type of product. 

SELECT visitor_sessions_day_id, 
COUNT(visitor_sessions_day_id) as count_visid, 
COUNT(distinct revenue) as count_rev_distinct,
COUNT(revenue) as count_rev_total,
COUNT(distinct unit_price) as count_unique_price


FROM public.analytics_distinct

GROUP BY visitor_sessions_day_id 
HAVING COUNT(distinct unit_price) < COUNT(revenue
ORDER BY count_visid DESC

--- I found that there are no cases where one visitor_sessions_day_id has more rows with revenues than unique unit_price's
--- further more the following code shows that there are no cases of a visit not including a unit price, so this must mean that the analytics table rows represents (amoung potentially other things) an interaction of type with a product by a user 
SELECT visitor_sessions_day_id, 
COUNT(visitor_sessions_day_id) as count_visid, 
COUNT(distinct revenue) as count_rev_distinct,
COUNT(revenue) as count_rev_total,
COUNT(distinct unit_price) as count_unique_price,
COUNT( unit_price) as has_price


FROM public.analytics_distinct

GROUP BY visitor_sessions_day_id 
HAVING COUNT( unit_price) != COUNT(visitor_sessions_day_id)
ORDER BY count_visid DESC

-- Does the visitid represent a page? What exactly does it mean 




