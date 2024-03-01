# QA Process
What are your risk areas? Identify and describe them.

## 1. QA for both tables (**all_sessions** and **analytics**) contain a similar range of dates in order to determine combination is valid or not.
**Describe the QA process:**
* By comparing the count of distinct dates between the 'all_sessions' and 'analytics' tables, it aims to ensure that both datasets align in terms of the date field.
* If the QA test passes, it indicates that the date data in both tables is consistent, reducing the risk of errors or inaccuracies in downstream processes. However, if the test fails, further investigation and corrective actions may be necessary to address any underlying data issues and maintain data quality and integrity.

**SQL query used to execute:**
```sql
-----QA using distinct value count of 'date' in 'all_sessions' and 'analytics' tables
--cte query
WITH cte_distinct_DATE_in_2tables AS(
		SELECT 'all_sessions' AS table_name, 
			   COUNT(*) AS count_rows, 
			   'date' AS key_name,
			   COUNT(DISTINCT date) AS count_distinct_date
		FROM all_sessions
			
		UNION ALL
		
		SELECT 'analytics' AS table_name,
			   COUNT(*) AS count_rows,    
			   'date' AS key_name,
			   COUNT(DISTINCT date) AS count_distinct_date  
		FROM analytics
        ),  
			
QA_raw AS(
	SELECT COUNT(DISTINCT count_distinct_date) AS test_result 
	FROM cte_distinct_DATE_in_2tables
	)
--main query
SELECT 
		'referential_integrity_check' AS qa_test_name,
       (CASE WHEN test_result = 1 THEN 'pass'
        	ELSE 'fail'
       END) AS qa_result
FROM 
		qa_raw;

```     
**Result:** 
A discrepancy in the count of distinct dates could signify missing data in one of the tables (in _analytics_ table), potentially leading to incomplete analysis or inaccurate insights.



## 2. QA for duplicates: to see if there are any rows duplicated in **all_session** table
**Describe the QA process:**
* By group by all columns in table, then count it to see if there are any rows duplicated.
* If QA query returns no records, it means there is no row duplicated. In contrast, it means table has duplicate row, then we need to address it/them by delete the duplicates.

**SQL query used to execute:**
```sql
SELECT COUNT(*) AS duplicate_count,
       fullvisitorid, channelgrouping, time, country, city, 
       totaltransactionrevenue, transactions, timeonsite, pageviews, 
	   session_quality_dim, date, visitid, type,
	   product_refund_amount, productquantity, productprice, productrevenue,
	   productsku, v2productname, v2productcategory, productvariant,
	   currencycode, itemquantity, itemrevenue, transactionrevenue, transactionid,
	   pagetitle, searchkeyword, pagepathlevel1,
	   ecommerceaction_type, ecommerceaction_step, ecommerceaction_option
FROM all_sessions
GROUP BY        fullvisitorid, channelgrouping, time, country, city, 
       totaltransactionrevenue, transactions, timeonsite, pageviews, 
	   session_quality_dim, date, visitid, type,
	   product_refund_amount, productquantity, productprice, productrevenue,
	   productsku, v2productname, v2productcategory, productvariant,
	   currencycode, itemquantity, itemrevenue, transactionrevenue, transactionid,
	   pagetitle, searchkeyword, pagepathlevel1,
	   ecommerceaction_type, ecommerceaction_step, ecommerceaction_option
HAVING COUNT(*) > 1;
```
**QA code_version 2:
```sql
SELECT 'QA_duplicate_rows' AS qa_test_name,
       (CASE WHEN (SELECT COUNT(*)
		   FROM (SELECT DISTINCT * FROM all_sessions)) --this returns 'unique_rows_count' 
		= (SELECT COUNT(*) FROM all_sessions) --this returns 'total_rows_count' 
		THEN 'pass'
        	ELSE 'fail'
       END) AS qa_result
```

**Result:** 
There is no duplicate rows in in **all_session** table.



## 3. QA to check completeness and correctness of 'fullvisitorid' field
**Describe the QA process:**
This QA code assesses the completeness and correctness of the 'fullvisitorid' field in the 'all_sessions' table to identifies if there is any potential issues that need further investigation and resolution.

**SQL query used to execute:**
```sql
SELECT COUNT(*) AS rows_count,
       COUNT(DISTINCT fullvisitorid) AS distinct_fullvisitorid_count,
       COUNT(CASE WHEN fullvisitorid IS NULL THEN 1 END) AS null_fullvisitorid_count
FROM all_sessions
```

**Result:**
* null_fullvisitorid_count: 0  -> no null values in **fullvisitorid** column.
* rows_count: 15134  -> table has 15134 rows.
* distinct_fullvisitorid_count: 14223  -> table has 14223 unique **fullvisitorid** values.
====> with the result from previous QA (QA 2), we can conclude **fullvisitorid** can access multiple times/sessions.
====> Should not delete fullvisitorid duplicated.

```sql
--Count visits by distinct fullvisitorid
SELECT fullvisitorid, COUNT(*)
FROM all_sessions
GROUP BY fullvisitorid
HAVING COUNT(*)>1
```

## 4. Check for any abnormal values in country and city columns
```sql
SELECT country, COUNT(*)
FROM all_sessions 
GROUP BY country
ORDER BY country
```

```sql
SELECT city, COUNT(*) 
FROM all_sessions 
GROUP BY city
ORDER BY city
```
**Result:**
 I do not have knowlege to address the abnormal values in _city_ and _country_ columns:
	* _country_ column has 24 abnormal values '(not set)'
	* _city_ column has 2 abnormal types: 354 '(not set)' values and 8302 'not available in demo dataset'.
I thought about replacing them by mode values, but I found it unreasonable.