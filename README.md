# Final-Project-Transforming-and-Analyzing-Data-with-SQL

## Project/Goals
Getting to know the data,
 -   what is each column likely to represent,
 -   in what context was the data likely collected
Cleaning Data
 - Where is data missing, and where is complete
 -   finding the most important columns which do not have missing data or hard to handle values
 -   filling in empty columns, where appropriate 
 - Finding duplicates
 -   find duplicates within tables and between tables
 - Data Conversion. Convert important columns to approriate datatypes to save time when querying and increase performace
 - Creating new tables with unique identifiers similar to the northwinds demo data set
Answering the Questions Creating New Questions
-  


## Process

**Identify Entities and Relationships**:
        Investigate the tables to identify unique identifiers, facts, and potential entities.
        Analyze visitid and fullvisitorid for duplicates and investigate further.
  **Alter Data Types**:
 - Change data types for certain columns, such as converting text to VARCHAR and converting date columns to DATE.
**Transform Data**
- Change the time column to time_on_page (minor assumption) from milliseconds to seconds (appropriate for interval datatype) 
 **Create New Tables for Distinct Rows:**
-   Create new tables ('visitor_session' and 'visitor_session_source') to store distinct rows and related information.

**Create a Transactions Table:**
        Clean and update data in the 'all_sessions' table to populate the 'transactions' table with relevant transaction information.
 **Create a Visitor Info Table:**
        Clean and update data in the 'all_sessions' table to populate the 'visitor_info' table with visitor information.Populate 'visitor_session_with_source' and 'visitor_session_pk' Tables with Transaction Data:
        Populate specific tables ('visitor_session_with_source' and 'visitor_session_pk') with transaction data from 'analytics_distinct' and 'transactions' tables.
   
## Results

 --Total Number of Unique Visitors: Determined that there are 130,345 unique visitors based on their fullVisitorID.

--Analyzed the total number of unique visitors by channel grouping, providing insights into the most common referring sites.

--Unique Products Viewed by Each Visitor: Found the unique product viewed by each visitor and identified the visitor who viewed the most products.

--Percentage of Visitors Making a Purchase: Computed the percentage of visitors who made a purchase during their session on the website. The data revealed that only 3.96% of visitor sessions included a sale.

--Average Time Spent on a Page Type: Most time is spend on the review order, payment and basket pages.

--Most Common Page Visitors Spend Time On: Identified the webpage users spend the most time on, providing insights into user behavior on the website.

--Referral Traffic with Zero Time on Site: Analyzed the most common referral sources for users who spent zero time on the site and compared it to the general trend of referrals.

--Overall Time Spent on Site Over Time: Analyzed the overall time spent on the site over time, providing insights into changes in user behavior across different periods.

--Improvement in Sales Over the Weekend: Examined whether there was an improvement in sales revenue over the weekend compared to weekdays. The data showed that there was a modest increase in sales revenue on weekdays compared to weekends.

## Challenges 
I found the documentation process difficult. In pressing forward and answering questions, I would notice something I missed in the cleaning step and have to return to that process.
Keeping assumptions about the data straight, and keeping what transformations I have done in front of mind. 


## Future Goals
-Look into creating a new id called productsaleid. that only included information of a productsku when it sold, for what price etc, what time etc. This would be seperate from the transactionid which would include information on the total sale.
-Look into if there is a way to link productsku with the analytics csv
- Investigate further what channel referral means exactly


