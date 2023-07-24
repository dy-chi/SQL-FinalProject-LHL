# Final-Project-Transforming-and-Analyzing-Data-with-SQL


##Project Goals

The main objectives of this project are as follows:

    Getting to know the data by understanding what each column represents and the context in which the data was collected.

    Cleaning the data by addressing missing values, identifying important columns without missing data, and filling in empty columns where appropriate.

    Finding and handling duplicate records within and between tables.

    Converting important columns to appropriate data types for better query performance.

    Creating new tables with unique identifiers similar to the Northwind demo data set.

    Answering specific questions and formulating new ones based on the data analysis.

##Process

    Identify Entities and Relationships:
        Investigate the tables to identify unique identifiers, facts, and potential entities.
        Analyze visitid and fullvisitorid for duplicates and investigate further.

    Alter Data Types:
        Change data types for certain columns, such as converting text to VARCHAR and date columns to DATE.

    Transform Data:
        Change the time column to time_on_page (assuming milliseconds to seconds) to use the interval datatype.

    Create New Tables for Distinct Rows:
        Create 'visitor_session' and 'visitor_session_source' tables to store distinct rows and relevant information.

    Create a Transactions Table:
        Clean and update data in the 'all_sessions' table to populate the 'transactions' table with transaction information.

    Create a Visitor Info Table:
        Clean and update data in the 'all_sessions' table to populate the 'visitor_info' table with visitor information.

    Populate 'visitor_session_with_source' and 'visitor_session_pk' Tables with Transaction Data:
        Populate specific tables with transaction data from 'analytics_distinct' and 'transactions' tables.

##Results

The project's results include various insights gained from the data analysis, such as:

- Determining the total number of unique visitors (130,345) based on their fullVisitorID.
- Analyzing the total number of unique visitors by channel grouping, revealing the most common referring sites.
- Identifying unique products viewed by each visitor and finding the visitor who viewed the most products.
- Computing the percentage of visitors making a purchase during their session on the website (3.96%).
- Analyzing the average time spent on specific page types and identifying the most common pages visitors spend time on.
- Exploring referral traffic with zero time on site and comparing it to the general trend of referrals.
- Analyzing overall time spent on the site over time to identify changes in user behavior across different periods.
- Examining sales improvement over the weekend compared to weekdays.

##Challenges

The project encountered challenges in documenting the process and keeping track of data assumptions and transformations. Adapting and revisiting the cleaning steps while answering questions added complexity.
Future Goals

##Future Goals
The project's future goals include exploring the creation of a new id called productsaleid, focusing on productSKU details when sold, and linking productSKU with the analytics CSV. Further investigation is needed to understand the exact meaning of the "channel referral" data field.


