Answer the following questions and provide the SQL queries used to find the answer.

    
## Question 1: Which cities and countries have the highest level of transaction revenues on the site?

SQL Queries:

```sql
SELECT country, city, 
	ROUND(SUM(cleaned_totalTransactionRevenue),2) AS SUM_TotalTransactionRevenue,
	ROUND(SUM(cleaned_transactionRevenue),2)     AS SUM_TransactionRevenue
FROM view_all_sessions
GROUP BY country, city
ORDER BY SUM_TotalTransactionRevenue DESC NULLS LAST, SUM_TransactionRevenue DESC NULLS LAST
LIMIT 1;
```
**Note:** 
Because I don't have enough information as well as domain knowlege in this field, I'm not sure what the real difference between _totalTransactionRevenue_ and _transactionRevenue_ is, even though I spent much time to explore the data. Finally, I decided to include 2 these columns to answer this question.

Answer:
<p align="center" style="margin-top: 20px; margin-bottom: 20px;">
<img width="80%" src="https://github.com/ThuyTran102/SQL-Project-FEB2024/blob/main/images/Output_Q1.png" alt="Outcome"></img>
</p>



## Question 2: What is the average number of products ordered from visitors in each city and country?

SQL Queries:

**VERSION 1:**
```sql
SELECT country, city, ROUND(AVG(cleaned_productquantity), 2) AS avg_products_ordered
FROM view_all_sessions
WHERE transactionid IS NOT NULL 
   OR cleaned_transactions > 0 
GROUP BY country, city
ORDER BY avg_products_ordered DESC NULLS LAST;
```

**VERSION 2: Suppose the 2 tables ('all_sessions' and 'analytics') are related to each other**
```sql
WITH cte_total_unitssold_in_each_visitor AS(
	SELECT country, city, s.fullvisitorid, SUM(unitssold) sum_products_ordered
	FROM view_all_sessions AS s
		LEFT JOIN analytics AS a ON s.fullvisitorid = a.fullvisitorid
	WHERE transactionid IS NOT NULL
	   OR cleaned_transactions > 0
	GROUP BY country, city, s.fullvisitorid
	)
  --main query
SELECT country, city, ROUND(AVG(sum_products_ordered), 2) AS avg_products_ordered
FROM cte_total_unitssold_in_each_visitor
GROUP BY country, city
ORDER BY avg_products_ordered DESC NULLS LAST;


--Why I say "suppose"? Because I checked for the relation bewtween tables as below code, 
--but I feel like the databases of 2 these tables are not matched (due to the period of time) 
SELECT date FROM all_sessions  --15134 rows
SELECT date FROM analytics     --4301122 rows
SELECT DISTINCT date FROM all_sessions ORDER BY date --database 366 continuous days From 2016-08-01 to 2017-08-01 ( ORDER BY date ASC/DESC)
SELECT DISTINCT date FROM analytics ORDER BY date  --database 93 continuous days From 2017-05-01 to 2017-08-01    (ORDER BY date ASC/DESC) 
--Therefore, if we join 2 these tables by fullvisitorid and calculate on 'unitssold' columns that might results in incorrect answer.
```


Answer:

_Output of version 1:_

<p align="center" style="margin-top: 20px; margin-bottom: 20px;">
<img width="60%" src="https://github.com/ThuyTran102/SQL-Project-FEB2024/blob/main/images/Output_Q2_v1.png" alt="Outcome"></img>
</p>

_Output of version 2:_

<p align="center" style="margin-top: 20px; margin-bottom: 20px;">
<img width="60%" src="https://github.com/ThuyTran102/SQL-Project-FEB2024/blob/main/images/Output_Q2_v2.png" alt="Outcome"></img>
</p>



## Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?


SQL Queries:

```sql
--List visits that made orders on site by productcategory (in each country and city)
SELECT country, city, v2productcategory, 
		COUNT(v2productcategory) AS num_orders,
		SUM(cleaned_productquantity) as num_products,
		SUM(cleaned_productrevenue) AS total_revenue
FROM view_all_sessions
WHERE cleaned_transactions > 0
	 OR transactionid is NOT NULL 
GROUP BY country, city, v2productcategory
ORDER BY total_revenue DESC NULLS LAST, num_products DESC NULLS LAST, num_orders DESC NULLS LAST;
```

Answer:

<p align="center" style="margin-top: 20px; margin-bottom: 20px;">
<img width="100%" src="https://github.com/ThuyTran102/SQL-Project-FEB2024/blob/main/images/Output_Q3.png" alt="Outcome"></img>
</p>



## Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?


SQL Queries:

**VERSION 1:**
```sql
--CTE: The top-selling product in each country
WITH cte_products_sold_in_each_country AS(
		SELECT v2productname, country,
			   SUM(cleaned_productquantity) AS number_of_products_sold,
			   RANK() OVER(PARTITION BY country
						   ORDER BY SUM(cleaned_productquantity) DESC NULLS LAST) AS product_rank
		FROM view_all_sessions
		WHERE transactionid is NOT NULL
			OR cleaned_transactions > 0
		GROUP BY v2productname, country
)
--The best-selling product in each country
SELECT * 
FROM cte_products_sold_in_each_country
WHERE product_rank = 1 
	AND number_of_products_sold > 0
ORDER BY number_of_products_sold DESC;
```


**VERSION 2: Suppose the 2 tables ('all_sessions' and 'analytics') are related to each other**
```sql
--CTE
WITH cte_total_unitssold_in_each_visitor AS(
		SELECT country, s.fullvisitorid, SUM(unitssold) as sum_products_ordered
		FROM view_all_sessions AS s
			LEFT JOIN analytics AS a ON s.fullvisitorid = a.fullvisitorid
		WHERE transactionid IS NOT NULL
		   OR cleaned_transactions >0
		GROUP BY country, city, s.fullvisitorid
		),
    cte_products_sold_in_each_country AS(
		SELECT v2productname, a.country,
			   SUM(sum_products_ordered) AS number_of_products_sold,
			   RANK() OVER(PARTITION BY a.country
						   ORDER BY SUM(sum_products_ordered) DESC NULLS LAST) AS product_rank
		FROM view_all_sessions AS a
		LEFT JOIN cte_total_unitssold_in_each_visitor USING(fullvisitorid)
		WHERE transactionid is NOT NULL
			OR cleaned_transactions > 0
		GROUP BY v2productname, a.country
)
--The best-selling product in each country
SELECT * 
FROM cte_products_sold_in_each_country
WHERE product_rank = 1
	AND number_of_products_sold > 0
ORDER BY number_of_products_sold DESC;
```


Answer:

_Output of version 1:_

<p align="center" style="margin-top: 20px; margin-bottom: 20px;">
<img width="70%" src="https://github.com/ThuyTran102/SQL-Project-FEB2024/blob/main/images/Output_Q4_v1.png" alt="Outcome"></img>
</p>

_Output of version 2:_

<p align="center" style="margin-top: 20px; margin-bottom: 20px;">
<img width="85%" src="https://github.com/ThuyTran102/SQL-Project-FEB2024/blob/main/images/Output_Q4_v2.png" alt="Outcome"></img>
</p>



## Question 5: Can we summarize the impact of revenue generated from each city/country?

SQL Queries:
```sql
SELECT 
    country,
    ROUND(SUM(cleaned_totaltransactionRevenue),2) AS total_transactionrevenue,
    ROUND(SUM(cleaned_totaltransactionRevenue) / (SELECT SUM(cleaned_totaltransactionRevenue) FROM view_all_sessions), 2) * 100
	   AS transactionrevenue_percentage
FROM view_all_sessions
GROUP BY country
ORDER BY total_transactionrevenue DESC NULLS LAST;
```

Answer:

<p align="center" style="margin-top: 20px; margin-bottom: 20px;">
<img width="60%" src="https://github.com/ThuyTran102/SQL-Project-FEB2024/blob/main/images/Output_Q5.png" alt="Outcome"></img>
</p>
