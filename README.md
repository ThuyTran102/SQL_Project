# Project-Transforming-and-Analyzing-Data-with-SQL

## Project Goals
The goal of this project is to extract, explore, transform and analyze data using SQL (PostgreSQL). The dataset provided contains information about e-commerce activities, and the aim is to extract insights regarding customer behavior, product performance, and revenue generation.

## Prject Process
### Step 1: Loading csv files into Database on PgAdmin 4.
### Step 2: Explore to understanding data
* Explore relationships between different tables
* Explore relationships between different variables
### Step 3: Cleaning and transforming data
* Identify duplicate values
* Identify and handle missing values/ null values
* Detect and address invalid or incorrect values
### Step 4: Analyzing data
* Analyze patterns in customer behavior and product sales by cities and countries
### Step 5: QAing data


## Results
(what I discovered this data could tell me and how I used the data to answer those questions)
* Identified that United States had the highest transaction revenues on the site during the period from 2016-08-01 to 2017-08-01.
* Evaluated of revenue generation trends across different regions.
* Identified the average number of products and pattern in the types (product categories) which were ordered from visitors in each city and country through website.


## Challenges 
(discuss challenges I faced in the project)
1. Do the tables really relates each other? 
	* This dataset is frustrating to understand. After a long time studying each variable in each table, I realized that it seemed that the 5 tables provided did not complement each other's data.

2. HOW TO EXACTLY DETERMINE WHETHER A PRODUCT WERE SOLD or A PRODUCT WERE ORDERED BY VISITORS IN THIS SITUATION?
	* _transactionid_ IS NOT null ??
	* _transactions_ IS NOT null (or > 0) ??
	* _productquantity_ IS NOT null (or > 0) ??   or _itemquantity_ (or > 0) ??
	* _productrevenue_  IS NOT null (or > 0)??    or _itemrevenue_ IS NOT null (or > 0)???
	* _transactionrevenue_ IS NOT null (or > 0)   or _totalTransactionRevenue_ IS NOT null (or > 0) ??
	* _ecommerceaction_type_ or _ecommerceaction_step_: These column can contain the purchase action type, for example, "purchase" or "checkout". I do not have enough domain knowlege in this field and do not have enough information (4 remaining tables still do not have enough information) to understand the meaning of the numbers/values in ‘ecommerceaction_type’ and ‘ecommerceaction_step’ columns  Therefore, I will not use these 2 columns to determine whether a product has been purchased or ordered by a visitor.

3. I don't have enough information as well as domain knowlege in this field, I'm not sure what the real difference between _totalTransactionRevenue_ and _transactionRevenue_ is, even though I spent much time to explore the data. Therefore, I decided to include both of 2 these columns to answer the question 1.


3. I do not have knowlege to address the abnormal values in _city_ and _country_ columns:
	* _country_ column has 24 abnormal values '(not set)'
	* _city_ column has 2 abnormal types: 354 '(not set)' values and 8302 'not available in demo dataset'.
I thought about replacing them by mode values, but I found it unreasonable.


## Future Goals
(what would you do if you had more time?)
1. If I had an opportunity to talk with those who are experts in this field, I would try to determine a right way (right features) to define whether a product were sold or a product were ordered by visitors.

2. If more time were available, I would do analysis of customer behavior metrics, such as page views and time spent on site.
