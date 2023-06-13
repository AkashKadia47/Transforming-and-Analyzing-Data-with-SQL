What issues will you address by cleaning the data?

By cleaning the data I will be addressing the following issues,

1) Type conversion
2) Missing/null values
3) Dropping empty columns
4) Renaming the columns
5) Removing duplicates

For Data Validation and Data normalization; <br>
I will adopt a systematic approach to clean each column individually. This will involve a comprehensive examination of the column values and their relationships with other data elements. Through this process, I will gain insights into the specific cleaning requirements for each column. This may include routine procedures such as **adjusting formats, correcting capitalization, and applying mathematical transformations**. For more complex cases, advanced techniques will be utilized as needed.


Queries:
Below, provide the SQL queries you used to clean your data.

<h1> 1) Cleaning data by table </h1>

<h2>  "all_session" Table </h2>

**1) Type conversion**

```sql
ALTER TABLE all_sessions 
  ALTER COLUMN fullvisitorid TYPE numeric USING fullvisitorid::numeric,
  ALTER COLUMN productprice TYPE decimal USING productprice:: decimal,
  ALTER COLUMN totaltransactionrevenue TYPE integer USING totaltransactionrevenue::integer,
  ALTER COLUMN transactions TYPE real USING transactions::real,
  ALTER COLUMN productrefundamt TYPE real USING productrefundamt::real,
  ALTER COLUMN productquantity TYPE real USING productquantity::real,
  ALTER COLUMN productrevenue TYPE numeric USING productrevenue::numeric,
  ALTER COLUMN itemquantity TYPE real USING itemquantity::real,
  ALTER COLUMN itemrevenue TYPE real USING itemrevenue::real;
```

**2) Missing/null values**
Identyfing empty columns

```sql
SELECT 	sum(productrefundamt) AS productrefundamt,
	sum(totaltransactionrevenue) AS totaltransactionrevenue,
	sum(transactions) AS transactions,
	sum(sessionqualitydim) AS sessionqualitydim,
	sum(productquantity) AS productquantity,
	sum(itemquantity) AS iteamquantity,
	sum(itemrevenue) AS itemrevenue
FROM all_sessions;
```

**3) Dropping empty columns**

```sql
ALTER TABLE all_sessions
  DROP COLUMN itemquantity,
  DROP COLUMN itemrevenue,
  DROP COLUMN productrefundamt,
  DROP COLUMN searchkeyword;
```

**4) Extra cleaning by columns**
Type conversion for time and timeonsite column

```sql
ALTER TABLE all_sessions
  ALTER COLUMN time TYPE TIME USING (TIME '00:00:00' + (interval '1 second' * time::integer)),
  ALTER COLUMN timeonsite TYPE TIME USING (TIME '00:00:00' + (interval '1 second' * timeonsite::integer));

UPDATE all_sessions
SET timeonsite = '00:00:00'
WHERE timeonsite IS NULL
```

<h3>To simplify the data cleaning process, I have taken the proactive approach of transferring all unwanted, random, or redundant values to null instead of dealing with them individually across multiple columns.</h3>

*STEP 1: transforming all redundant value to null*

**Cleaning "productprice" column**

```sql
UPDATE all_sessions
SET productprice = ROUND(productprice::numeric, 2);

UPDATE all_sessions
SET productprice = productprice/1000000;

UPDATE all_sessions
SET productprice = ROUND(productprice::numeric, 2);
```

**Cleaning "city" and "country" columns**
```sql
UPDATE all_sessions
SET city = NULL
WHERE city = '(not set)';

UPDATE all_sessions
SET country = NULL
WHERE country = '(not set)';

UPDATE all_sessions
SET city = NULL
WHERE city = 'not available in demo dataset';
```

**Cleaning "v2productcategory"**
```sql
UPDATE all_sessions
SET v2productcategory = NULL
WHERE v2productcategory = '(not set)';
```

**Cleaning "currencycode"**
```sql
UPDATE all_sessions
SET currencycode = 'USA'
WHERE currencycode IS NULL; 
```

**Cleaning "totaltransactionrevenue" column**
```sql
ALTER TABLE all_sessions
  ALTER COLUMN totaltransactionrevenue TYPE decimal USING totaltransactionrevenue::decimal;
  Alter COLUMN productprice TYPE decimal USING productprice:: decimal,

UPDATE all_sessions
SET totaltransactionrevenue = ROUND(totaltransactionrevenue::numeric, 2);

UPDATE all_sessions
SET totaltransactionrevenue = totaltransactionrevenue/1000000;

UPDATE all_sessions
SET totaltransactionrevenue = ROUND(totaltransactionrevenue::numeric, 2);
```

*Step 2: changing null values to appropriate values*

```sql
UPDATE all_sessions
SET country = COALESCE(country, 'Not Provided'),
    city = COALESCE(city, 'Not Provided');

UPDATE all_sessions
SET totaltransactionrevenue = COALESCE(totaltransactionrevenue, '0'),
	sessionqualitydim = COALESCE(sessionqualitydim, '0'),
	transactions = COALESCE(transactions, '0'),
	productquantity = COALESCE(productquantity, '0'),
	productrevenue = COALESCE(productrevenue, '0')
```

<h2>Advance cleaning</h2>

**Data Validation of "productrevenue" column**

Cleaning the values in the "productrevenue" column that do not satisfy the calculation (productquantity * productprice).
```sql
UPDATE all_sessions
SET productrevenue = (productquantity * productprice)
WHERE productrevenue != (productquantity * productprice);

UPDATE all_sessions
SET productrevenue = ROUND(productrevenue::numeric, 2);
```

**Data Validation of "transactionrevenue" column**

```sql
SELECT totaltransactionrevenue, transactionrevenue, transactions, transactionid
FROM all_sessions

SELECT *
FROM products
UPDATE all_sessions
SET transactionrevenue = ROUND(transactionrevenue::numeric, 2)

UPDATE all_sessions
SET transactionrevenue = transactionrevenue/1000000;

--Updating the values in the "transactionrevenue" column that do not satisfy the calculation (totaltransactionrevenue / transactions).

UPDATE all_sessions
SET transactionrevenue = totaltransactionrevenue / transactions
WHERE transactions <> '0'
```

**Capitalization Normalization of the "type" column**
```sql
UPDATE all_sessions
SET type = CONCAT(UPPER(SUBSTRING(type, 1, 1)), LOWER(SUBSTRING(type, 2)));
```

**Data Validation of "v2productcategory" column**
```sql
--Checking for nulls.

SELECT t1.productsku, t1.v2productname, t1.v2productcategory AS null_category, t2.v2productcategory AS non_null_category
FROM all_sessions t1
JOIN all_sessions t2 ON t1.v2productname = t2.v2productname
WHERE t1.v2productcategory IS NULL
  AND t2.v2productcategory IS NOT NULL;

--Updating the information by identifying categories in different rows despite null values in the column.**

UPDATE all_sessions t1
SET v2productcategory = t2.v2productcategory
FROM all_sessions t2
WHERE t1.v2productcategory IS NULL
  AND t2.v2productcategory IS NOT NULL;
```

** Cleaning "ecommerceaction_type", "ecommerceaction_step", and "ecommerceaction_option"**

```sql
SELECT ecommerceaction_type, ecommerceaction_step, ecommerceaction_option
FROM all_sessions
WHERE ecommerceaction_option IS NULL

UPDATE all_sessions
SET ecommerceaction_option = 'Not Applicable'
WHERE ecommerceaction_option IS NULL
```

<h2> "analytics" table</h2>

**Cleaning "fullvisitorid" column**

```sql
ALTER TABLE analytics
  ALTER COLUMN fullvisitorid TYPE NUMERIC USING fullvisitorid::numeric;
```sql
	
**Cleaning "unitprice" column**

```sql
UPDATE analytics 
SET unitprice = (unitprice/1000000);

UPDATE analytics
SET unitprice = ROUND(unitprice::numeric, 2);
```

**Cleaning "timeonsite" column**

```sql
ALTER TABLE analytics
  ALTER COLUMN timeonsite TYPE TIME USING (TIME '00:00:00' + (interval '1 second' * timeonsite::integer));
```

**Checking and cleaning null values**
```sql
SELECT *
FROM analytics 
WHERE visitNumber IS NULL
	OR visitId IS NULL
	OR visitStartTime IS NULL
	OR date IS NULL
	OR fullvisitorId IS NULL
	OR channelGrouping IS NULL
	OR socialEngagementType IS NULL
	OR units_sold IS NULL
	OR pageviews IS NULL
	OR timeonsite IS NULL
	OR bounces IS NULL
	OR revenue IS NULL
	OR unitprice IS NULL;
```
**"unitsold" has nulls and *negetive* amount**
```sql
UPDATE analytics
SET units_sold = COALESCE(units_sold, 0) * -1
WHERE units_sold IS NULL OR units_sold < 0;
```

**"pageviews" has NULL**
```sql
SELECT pageviews, count(pageviews)
FROM analytics
GROUP BY pageviews;

UPDATE analytics
SET pageviews = '0'
WHERE pageviews IS NULL;
```

**"timeonsite" has NULL**
```sql
SELECT timeonsite, count(timeonsite)
FROM analytics
GROUP BY timeonsite;

UPDATE analytics
SET timeonsite = '00:00:00'
WHERE timeonsite IS NULL;
```
**"bounces" has NULL**
```sql
SELECT bounces, SUM(bounces)
FROM analytics
GROUP BY bounces;

UPDATE analytics
SET bounces = '0'
WHERE bounces IS NULL;
```

**"revenue" has NULL**
```sql
SELECT revenue, count(revenue)
FROM analytics
GROUP BY revenue;

UPDATE analytics
SET revenue = '0'
WHERE revenue IS NULL;
```

<h2> "products" Table </h2>

**Renaming the "sku" column to facilitate the use of the join function.**
```sql
ALTER TABLE products
RENAME COLUMN sku TO productsku;
```
**Set primary kay**
```sql
ALTER TABLE products
ADD PRIMARY KEY (productsku);
```
**Cleaning null values**
```sql
UPDATE products
SET sentimentscore = COALESCE(sentimentscore, 0),
    sentimentmagnitude = COALESCE(sentimentmagnitude, 0)
WHERE sentimentscore IS NULL OR sentimentmagnitude IS NULL;
```

<h3>Advance cleaning</h2>
**Trimming the leading spaces in the values of the "name" column.**

```sql
UPDATE products
SET name = LTRIM(name);
```

<h2>"sales_by_sku" Table</h2>

**Assigning Primary Key**
```sql
ALTER TABLE sales_by_sku
ADD PRIMARY KEY (productsku);
```

**Removing rows with null values or duplicate values.**

```sql
DELETE FROM sales_by_sku
WHERE total_ordered = 0;
```

<h2> "sales_report" Table </h2>

**Assigning Primary Key**
```sql
ALTER TABLE sales_report
ADD PRIMARY KEY (productsku);
```

**Cleaning "ratio" column**
```sql
UPDATE sales_report
SET ratio = '0'
WHERE ratio IS NULL;

--Cleaning it further

UPDATE sales_report
SET ratio = CAST(ratio AS numeric(10, 3))
WHERE ratio IS NOT NULL;
```
