Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:
       
    SELECT city, country, SUM(totaltransactionrevenue) AS transactionrevenue
    FROM all_sessions
    GROUP BY city, country
    ORDER BY city DESC , country DESC
    LIMIT 1;

Answer:

City: Zurich

Country: Switzerland

Transactionrevenue total: 16990000

**Question 2: What is the average number of products ordered from visitors in each city and country?**

SQL Queries:

    SELECT als.city, als.country, AVG(p.orderedquantity) AS avg_product_ordered
    FROM products p
    JOIN all_Sessions als USING(productsku)
    GROUP BY als.city, als.country
    ORDER BY als.city, als.country;

    --or rounding the avg

    SELECT als.city, als.country, ROUND(AVG(p.orderedquantity)) AS avg_product_ordered
    FROM products p
    JOIN all_Sessions als USING(productsku)
    GROUP BY als.city, als.country
    ORDER BY als.city, als.country;
    
    --total_average_product_ordered 
    
    SELECT AVG(orderedquantity) AS total_avg_product_ordered
    FROM
	(
		SELECT als.city, als.country, p.orderedquantity
		FROM products p
		JOIN all_Sessions als USING(productsku)
	) AS asd

Answer:

Average of total products ordered from each city and countries : 517.3914968323028776


**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:



Answer:





**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:



Answer:





**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:



Answer:







