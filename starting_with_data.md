## Question 1: Which are the top 10 countries that have the most website visits during the period from 2016-08-01 to 2017-08-01?

SQL Queries:
```sql
SELECT country, COUNT(*) AS number_of_visits
FROM view_all_sessions 
GROUP BY country
ORDER BY number_of_visits DESC
LIMIT 10;
```
**Note:** Here, I do not use **COUNT(DISTINCT fullvisitorid)** because I want to count visits (visit times). One fullvisitorid can visit the website many times.

Answer:

<p align="center" style="margin-top: 20px; margin-bottom: 20px;">
<img width="35%" src="https://github.com/ThuyTran102/SQL-Project-FEB2024/blob/main/images/Output_ownQ1.png" alt="Outcome"></img>
</p>


## Question 2: In each country, how many website visits resulted in transactions during the period from 2016-08-01 to 2017-08-01?

SQL Queries:
```sql
SELECT country, COUNT(*) AS transactions_count
FROM view_all_sessions
WHERE cleaned_transactions > 0 OR transactionid is NOT null
GROUP BY country;
```

Answer:

<p align="center" style="margin-top: 20px; margin-bottom: 20px;">
<img width="40%" src="https://github.com/ThuyTran102/SQL-Project-FEB2024/blob/main/images/Output_ownQ2.png" alt="Outcome"></img>
</p>


## Question 3: List all product names along with their sku code that are ordered within the countries of United States or Canada during the period from 2016-08-01 to 2017-08-01.

SQL Queries:
```sql
SELECT DISTINCT(productsku), productname
FROM view_all_sessions AS v
LEFT JOIN products AS p ON v.productsku = p.sku 
WHERE LOWER(country) IN ('united states', 'canada')
		AND (cleaned_transactions > 0  OR transactionid is NOT null  OR productquantity > 0);
```

Answer:

<p align="center" style="margin-top: 20px; margin-bottom: 20px;">
<img width="50%" src="https://github.com/ThuyTran102/SQL-Project-FEB2024/blob/main/images/Output_ownQ3.png" alt="Outcome"></img>
</p>



## Question 4: Get the growth/difference of website visits resulting in transactions in 2017 compared to 2016 in each contry.

SQL Queries:
```sql
WITH cte_transactions_count_2017_by_country AS(
		SELECT country, COUNT(*) AS transactions_count_2017
		FROM view_all_sessions
		WHERE cleaned_transactions > 0 AND EXTRACT(year FROM date) = 2017
		GROUP BY country
		),
	 cte_transactions_count_2016_by_country AS(
		SELECT country, COUNT(*) AS transactions_count_2016
		FROM view_all_sessions
		WHERE cleaned_transactions > 0 AND EXTRACT(year FROM date) = 2016
		GROUP BY country
		)
SELECT 	country, 
	   	transactions_count_2017, 
		COALESCE (transactions_count_2016, 0) AS transactions_count_2016,
		(transactions_count_2017 - COALESCE (transactions_count_2016, 0)) AS transaction_difference
FROM cte_transactions_count_2017_by_country
	FULL JOIN cte_transactions_count_2016_by_country USING(country);
```

Answer:

<p align="center" style="margin-top: 20px; margin-bottom: 20px;">
<img width="80%" src="https://github.com/ThuyTran102/SQL-Project-FEB2024/blob/main/images/Output_ownQ4.png" alt="Outcome"></img>
</p>


