## What issues will I address by cleaning the data?
In *all_sessions* table, I clean data that I need for answering the later questions as follows:

1. I clean data in _productprice_ column by dividing by 1,000,000 (because I saw _currencycode_ containing all 'USD' values and products are normal items, so it is invalid when unit price per product is millions US dollar.
2. I address null values in _productquantity_ and _transactions_ columns, and I also make sure cho detect and change incorrect values in numeric columns by replacing non-numeric values with NULL, then replacing NULL with 0. This ensures that the column contains only numeric values, helps to address potential inconsistencies or errors for analysis; for example, answering the question about avarge number of products ordered from visitors in a right way.
3. I address null values in _transactionrevenue_, _totaltransactionrevenue_, and _productrevenue_ columns similarly to above. 
	1. Detect if value contains any character which is  different number 0-9 and then replace non-numeric values with NULL
	2. Then repalce all exsiting NULL values with 0 (keep if values are valid numbers)
	3. Then divide by 1,000,000 in order to suit with the real sistuation


**Queries:**
```sql
CREATE OR REPLACE VIEW view_all_sessions AS
	SELECT *,
			--1.Clean data in 'productprice' column by dividing by 1,000,000
			productprice / 1000000 As cleaned_productprice,
			
			--2.Clean data in 'productquantity' column	
			(CASE WHEN (productquantity::VARCHAR) LIKE '[^0-9]' THEN NULL
	  	   		 WHEN productquantity IS NULL THEN 0
	  	   		 ELSE productquantity
	  		END) AS cleaned_productquantity,
			
			--3.Clean data in 'transactions’ column	
			(CASE WHEN (transactions::VARCHAR) LIKE '[^0-9]' THEN NULL
	  	   		 WHEN transactions IS NULL THEN 0
	  	   		 ELSE transactions
	  		END)  AS cleaned_transactions,	
			
			--4.Clean data in 'transactionrevenue' column
			(CASE WHEN (transactionrevenue::VARCHAR) LIKE '[^0-9]' THEN NULL
	  	   		 WHEN transactionrevenue IS NULL THEN 0
	  	   		 ELSE transactionrevenue
	  		END) / 1000000 AS cleaned_transactionrevenue,	
			
			--5.Clean data in 'totaltransactionrevenue' column	
			(CASE WHEN (totaltransactionrevenue::VARCHAR) LIKE '[^0-9]' THEN NULL
	  	   		 WHEN totaltransactionrevenue IS NULL THEN 0
	  	   		 ELSE totaltransactionrevenue
	  		END) / 1000000 AS cleaned_totaltransactionrevenue,		
			
			--5.Clean data in 'productrevenue' column	
			(CASE WHEN (productrevenue::VARCHAR) LIKE '[^0-9]' THEN NULL
	  	   		 WHEN productrevenue IS NULL THEN 0
	  	   		 ELSE productrevenue
	  		END) / 1000000 AS cleaned_productrevenue		
	FROM all_sessions;
```

**If we want to update the original database, use the following code:**
```sql
UPDATE all_sessions
SET 
    productprice = productprice / 1000000,
    productquantity = CASE 
                          WHEN (productquantity::VARCHAR) LIKE '[^0-9]' THEN NULL
                          WHEN productquantity is NULL THEN COALESCE(productquantity, 0)
			  ELSE productquantity
                      END,
    transactions = CASE 
                      WHEN (transactions::VARCHAR) LIKE '[^0-9]' THEN NULL
                      WHEN transactions is NULL THEN COALESCE(transactions, 0)
		      ELSE transactions
                  END,
    totaltransactionrevenue = CASE 
                                  WHEN (totaltransactionrevenue::VARCHAR) LIKE '[^0-9]' THEN NULL
				  WHEN totaltransactionrevenue is NULL THEN COALESCE(totaltransactionrevenue, 0)
                                  ELSE totaltransactionrevenue / 1000000
                              END,
    transactionrevenue = CASE 
                             WHEN (transactionrevenue::VARCHAR) LIKE '[^0-9]' THEN NULL
			     WHEN transactionrevenue is NULL THEN COALESCE(transactionrevenue, 0)
                             ELSE transactionrevenue / 1000000
                         END;
```