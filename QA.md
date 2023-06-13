What are your risk areas? Identify and describe them.

Several risk areas were identified and addressed throughout the project to ensure data quality in the analytics process.
These included risks related to;
1) Data accuracy 
2) Data completeness
3) Data transformation. 

Challenges such as inconsistent column names, duplicates, empty columns, and capitalization normalization were also encountered. Additionally, mathematically incorrect columns required careful examination and correction. 

The project successfully delivered reliable and meaningful analytics outcomes by addressing these risks through rigorous cleaning, standardization, and verification processes.


QA Process:
Describe your QA process and include the SQL queries used to execute it.


**1) For a comprehensive description of data cleaning activities related to data accuracy, completeness, and transformation, please refer to the "cleaning_data.md" file, where detailed information regarding the proper cleaning and handling of values can be found.**

**2) Complex QA process**

*Data Validation of "all_sessions.productrevenue" column*

To ensure accurate analysis results, transform the values in the "all_sessions.productrevenue" column that does not align with the calculation (all_sessions.productquantity * all_sessions.productprice). Given the significant role of product revenue in the analysis process, this query aims to mathematically normalize the values, thereby mitigating the risk of inaccurate conclusions.
```sql
UPDATE all_sessions
SET productrevenue = (productquantity * productprice)
WHERE productrevenue != (productquantity * productprice);
```

Similar to the transformation performed for the "all_sessions.productrevenue" column, the values in the "all_sessions.totaltransactionrevenue" column were also adjusted to align with the calculation logic.
```sql
UPDATE all_sessions
SET transactionrevenue = totaltransactionrevenue / transactions
WHERE transactions <> '0'
```

*Capitalization Normalization of the "all_sessions.type" column* 
```sql
UPDATE all_sessions
SET type = CONCAT(UPPER(SUBSTRING(type, 1, 1)), LOWER(SUBSTRING(type, 2)));
```

*Data Validation of "all_sessions.v2productcategory" column*

Updating the information by identifying categories in different rows despite null values in the column.
```sql
UPDATE all_sessions t1
SET v2productcategory = t2.v2productcategory
FROM all_sessions t2
WHERE t1.v2productcategory IS NULL
  AND t2.v2productcategory IS NOT NULL;
```  
  
*3) It has been observed that the "all_sessions.transactionid" column contains inaccurate information in some rows, where values from the "all_sessions.transactionrevenue" and "all_sessions.transactions" columns are present bit not in "all_sessions.transactionid".*

To address this issue, a logical approach for generating the transaction ID is proposed. The transaction ID should contain 15 characters, starting with "ORD" followed by the ordered date and a four-character code. However, determining the last four characters accurately from the available tables presents challenges. This make the cleaning process for this particular column challenging and potentially impossible.
```sql
SELECT transactionid
FROm all_sessions
WHERE transactionid <> 'Not Applicable'
```
