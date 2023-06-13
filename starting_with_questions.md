Answer the following questions and provide the SQL queries used to find the answer.

    
<h2>Question 1: Which cities and countries have the highest level of transaction revenues on the site?</h2>

SQL Queries:
```sql
SELECT city, country, SUM(totaltransactionrevenue) AS transactionrevenue
FROM all_sessions
WHERE CITY IS NOT NULL
GROUP BY city, country
HAVING SUM(totaltransactionrevenue) > 0
ORDER BY transactionrevenue DESC
LIMIT 15;
```
Answer: <br>
![Ans 1](https://github.com/AkashKadia47/SQL-Project_1/assets/114313486/42bcaf84-43e3-4252-abb1-e215975bd59a)<br>
Based on the available data, it appears that the highest transaction-generated country is the USA. However, the city data encompasses all cities, including those where customers have not provided values. Therefore, relying on this data alone to accurately determine the city with the highest transaction revenue is not feasible. <br>
Nevertheless, it is reasonable to conclude that San Francisco, located in the United States, exhibits the highest transaction revenue. By setting a limit of 15, it becomes evident that the United States leads in terms of revenue generated through transactions. Additionally, it is noteworthy that Tel Aviv-Yafo in Israel ranks significantly, securing the sixth position with a transaction revenue of 602.00. Furthermore, Sydney in Australia demonstrates a revenue of 358.00.

<h2>Question 2: What is the average number of products ordered from visitors in each city and country?</h2>

SQL Queries:

The question is unclear, I understand it in a couple of ways.

**1) Average of products ordered by each countries and cities.**

*Solution*
```sql
--- Query 1
SELECT als.city, als.country, AVG(p.orderedquantity) AS avg_product_ordered
FROM products p
JOIN all_Sessions als USING(productsku)
GROUP BY als.city, als.country
ORDER BY als.city, als.country;

--or rounding the avg
-- Query 2

SELECT als.city, als.country, ROUND(AVG(p.orderedquantity)) AS avg_product_ordered
FROM products p
JOIN all_Sessions als USING(productsku)
GROUP BY als.city, als.country
ORDER BY als.city, als.country;
```

**2: Average of total ordered products from everycountry, this is not just average of ordered products, I have counted this with respect to cities and countries.**
```sql
--- Query 3
SELECT AVG(orderedquantity) AS total_avg_product_ordered
FROM
	(
		SELECT als.city, als.country, p.orderedquantity
		FROM products p
		JOIN all_Sessions als USING(productsku)
	) AS asd;
	
    SELECT als.city, als.country, AVG(p.orderedquantity) AS avg_product_ordered
    FROM products p
    JOIN all_Sessions als USING(productsku)
    GROUP BY als.city, als.country
    ORDER BY als.city, als.country;
```
Answer:

Query 1:<br>
![Ans 2_1](https://github.com/AkashKadia47/SQL-Project_1/assets/114313486/561a6469-c5cf-489f-9ad9-0d2299d36e2d)<br>
Query 2:<br>
![Ans 2_2](https://github.com/AkashKadia47/SQL-Project_1/assets/114313486/ecd8a9e1-50a2-4ad0-8c4e-92b5fbf5be43)<br>
Query 3:<br>
![Ans 2_3](https://github.com/AkashKadia47/SQL-Project_1/assets/114313486/49051033-6896-4cc1-a54d-67460c880a67)

<h2>Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?</h2>

SQL Queries With answer:

To accurately count only the quantity of products that have been ordered (with a quantity greater than zero), it is necessary to perform a join operation between the "all_sessions" and "products" tables. This ensures that any products that have not been ordered are excluded from the count, resulting in a more precise representation of the ordered product quantities.

```sql
SELECT als.city, als.country, als.v2productcategory, count(*) total_orders
FROM all_sessions als
JOIN products p USING(productsku)
WHERE p.orderedquantity > 0
GROUP BY als.city, als.country, als.v2productcategory
ORDER BY total_orders DESC;
```
![Ans 3_1](https://github.com/AkashKadia47/SQL-Project_1/assets/114313486/c887c548-7537-4781-b582-1eabd6b6c914)<br>
Based on the available data, it is evident that the United States has the highest number of customers placing orders on the website, with a significant preference for the home category. Popular products among these customers include men's T-shirts, electronics, and apparel.<br>

However, it is worth noting that the city column lacks proper values provided by the customers, making it difficult to determine the specific city from which the highest number of orders originate. To address this limitation and gain more accurate insights, we can execute the following query:
```sql
SELECT als.city, als.country, als.v2productcategory, count(*) total_orders
FROM all_sessions als
JOIN products p USING(productsku)
WHERE p.orderedquantity > 0 AND city <> 'Not Provided'
GROUP BY als.city, als.country, als.v2productcategory
ORDER BY total_orders DESC;
```
![Ans 3_2](https://github.com/AkashKadia47/SQL-Project_1/assets/114313486/05ec5eea-16f6-4d90-b711-3f6cca9b8802)<br>
The result shows that Mountain View, within the United States, stands out as the city with the highest number of customers, especially for men's apparel, with over 117 orders. Another city worth targeting for men's apparel is New York City. Understanding these trends enables businesses to effectively tailor their marketing strategies to reach the desired audience.

<h2>Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?</h2>

SQL Queries and Answer:
```sql
SELECT country, city, name, orderedquantity
FROM (
    SELECT a.city, a.country, p.name, SUM(p.orderedquantity) as orderedquantity,
           ROW_NUMBER() OVER (PARTITION BY a.city, a.country ORDER BY SUM(p.orderedquantity) DESC) as rank
    FROM all_sessions a
	JOIN products p USING(productsku)
    GROUP BY city, country, name
) ranked_data
WHERE rank = 1
GROUP By country, city, name, orderedquantity
ORDER BY orderedquantity DESC, country;
```
![Ans 4_1](https://github.com/AkashKadia47/SQL-Project_1/assets/114313486/861c3c2b-6711-4854-bc4c-0687bf2d3f4d)<br>
Despite the absence of customer-provided city information, it is interesting to observe that the highest amount of Custom Decals is generated from the United States.<br>

To address this limitation and gain more accurate insights, we can execute the following query:
```sql
SELECT country, city, name, orderedquantity
FROM (
    SELECT a.city, a.country, p.name, SUM(p.orderedquantity) as orderedquantity,
           ROW_NUMBER() OVER (PARTITION BY a.city, a.country ORDER BY SUM(p.orderedquantity) DESC) as rank
    FROM all_sessions a
	JOIN products p USING(productsku)
    GROUP BY city, country, name
) ranked_data
WHERE rank = 1 AND city <> 'Not Provided'
GROUP By country, city, name, orderedquantity
ORDER BY orderedquantity DESC, country;
```
![Ans 4_2](https://github.com/AkashKadia47/SQL-Project_1/assets/114313486/a394c722-df77-4e55-aaad-c4f2bd54e478)<br>
The highest-selling product overall is the Kick Ball. The United States emerges as the country with the highest number of products sold, including the Kick Ball and the cam Outdoor Security Camera. Mountain View is the city with the highest number of ordered products, with an impressive 75,850 Kick Balls. The output provides information on the highest-ordered items by countries and cities.

<h2>Question 5: Can we summarize the impact of revenue generated from each city/country?</h2>

SQL Queries:
```sql
SELECT als.city, als.country, p.name, SUM(als.productprice * p.orderedquantity) AS total_revenue
FROM all_sessions als
JOIN products p USING(productsku)
WHERE city IS NOT NULL
GROUP BY als.city, als.country, p.name
ORDER BY total_revenue DESC, country, city
```
Answer:<br>
![Ans 5](https://github.com/AkashKadia47/SQL-Project_1/assets/114313486/b35f9eb2-320f-463d-b98b-b9bfd137941b)<br>
The highest revenue is generated by the United States at the country level, with Mountain View as the leading city. The products driving this success are the Cam Indoor Security Camera - USA and the Cam Outdoor Security Camera - USA. Additionally, when analyzing the data, it becomes apparent that a significant portion of orders originate from the United States, followed by the United Kingdom, Colombia, India, and other countries. The data suggests that the highest-selling products belong to the Home category.


