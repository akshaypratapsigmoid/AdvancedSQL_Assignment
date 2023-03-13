# Peer_Learning_Document

## Mahesh's Approach

### QUESTION 1:
Write a query that gives an overview of how many films have replacements costs in the following cost ranges
low: 9.99 - 19.99
medium: 20.00 - 24.99
high: 25.00 - 29.99
#### CODE:

SELECT
SUM(
CASE
WHEN replacement_cost between 9.99 and 19.99  THEN 1 ELSE 0 END
)  AS 'no_of_low' ,
SUM(
CASE
WHEN replacement_cost between 20.00 and 24.99  THEN 1 ELSE 0 END
)  AS 'no_of_medium' ,
SUM(
CASE
WHEN replacement_cost between 25.00 and 29.99  THEN 1 ELSE 0 END
)  AS 'no_of_high'
FROM film

##### Approach : 
+ Here in this particular question we have been asked to give the overview of the films where the replacement_cost is in the given range categorizing it into low,medium and high
+ Logic in the case statement is if the particular replacement_cost is within the specified range for the particular category then returning 1 else 0 which we are summing up using SUM() function which results in total number of films in that particular range


##### OUTPUT :

<img width="441" alt="Screenshot 2023-03-08 at 2 11 38 PM" src="https://user-images.githubusercontent.com/122472996/223664760-6613db6b-21d0-4c2c-80ee-180e3cbf0f4c.png">

### QUESTION 2:
Write a query to create a list of the film titles including their film title, film length and film category name ordered descendingly by the film length. Filter the results to only the movies in the category 'Drama' or 'Sports'.
+ "STAR OPERATION" "Sports" 181
+ "JACKET FRISCO" "Drama" 181
#### CODE:

SELECT title AS 'Film Title' , 
c.name AS 'Film Category', 
length_ AS 'Film Length' 
FROM film f JOIN film_category fc
ON f.film_id = fc.film_id
JOIN category c 
ON fc.category_id = c.category_id
WHERE c.name = 'Drama' or c.name='Sports' 
ORDER BY f.length_ DESC ;

##### Approach :  
 + Here we have to create a list of the film titles including their film title, film length and film category name ordered descendingly by the film length for the category “sports” and “drama”.
+ So for that I have joined three tables i.e., film, film_category, category and extracted the film_title, film_length and film_category by applying the filter for category name to be only “Sports” and “Drama” ordering by the length of the film in descending order.


##### OUTPUT : 
<img width="585" alt="Screenshot 2023-03-08 at 2 16 07 PM" src="https://user-images.githubusercontent.com/122472996/223665771-b6aac043-abe7-47ff-b497-e923908993e9.png">

### QUESTION 3:
Write a query to create a list of the addresses that are not associated to any customer.

#### CODE:

SELECT a.address_id ,
a.address
FROM customer c 
RIGHT JOIN address a
ON c.address_id = a.address_id
WHERE c.customer_id IS NULL

##### Approach :
+ We have been asked to create a list of the addresses that are not associated to any customer
+ So For that I have used two tables i.e., `address` and `customer`.
+ I have used right join above to retrieve all the data from the address table irrespective of whether it is present in the customer table. So whichever customer_id is not associated with any address will result in null values which i am using in the where clause to filter out the data where customer_id is null which will give me all the address which are not related to any customer.


##### OUTPUT : 
<img width="585" alt="Screenshot 2023-03-08 at 2 19 26 PM" src="https://user-images.githubusercontent.com/122472996/223666541-33b758d1-0e0a-4340-bcb3-2994a7e688e8.png">


### QUESTION 4:
Write a query to create a list of the revenue (sum of amount) grouped by a column in the format "country, city" ordered in decreasing amount of revenue.
* eg. "Poland, Bydgoszcz" 52.88

#### CODE:

SELECT concat(cty.country,' , ' ,ct.city  ) AS country_city ,
SUM(py.amount)  AS 'List of the revenue'
FROM rental r 
JOIN customer c
ON r.customer_id = c.customer_id
JOIN address a 
ON c.address_id = a.address_id
JOIN city ct 
ON a.city_id = ct.city_id
JOIN country cty
ON cty.country_id = ct.country_id
JOIN payment py 
ON r.rental_id = py.rental_id
GROUP BY cty.country, ct.city 

##### Approach: 
+ Here we need to create a list of the revenue based on country, city according to decreasing amount of revenue
+ So for getting the desired result I have joined 4 tables i.e., country, city, address and payment to fetch country name, city name which I am concatenating using CONCAT() function
+ Also calculated the revenue by using window function which is calculating the sum of amount partitioning by Country,City and then ordering revenue by descending order
##### OUTPUT :
<img width="599" alt="Screenshot 2023-03-08 at 2 24 02 PM" src="https://user-images.githubusercontent.com/122472996/223667564-13bd53f8-cb88-4ee6-8c42-b0ae74aee6bc.png">

### QUESTION 5:
Write a query to create a list with the average of the sales amount each staff_id has per customer.
result: 
+ 2 56.64
+ 1 55.91

#### CODE:

SELECT staff_id, ROUND(AVG(sum_amount), 2) AS sales_amount
FROM (
SELECT DISTINCT staff_id, customer_id, 
SUM(amount) OVER(PARTITION BY staff_id,customer_id) AS sum_amount 
FROM payment 
)a
GROUP BY staff_id

##### Approach :
+ Here we are asked to create a list with the average of sales per customer with each staff id.
+ In this I have used the window function to calculate the sum_amount 
##### OUTPUT :
<img width="599" alt="Screenshot 2023-03-08 at 2 26 51 PM" src="https://user-images.githubusercontent.com/122472996/223668179-13554356-959b-4726-bcce-986f70cbb1e0.png">

### QUESTION 6:
Write a query that shows average daily revenue of all Sundays.

#### CODE:

SELECT ROUND(avg(am),2) AS 'average daily revenue of all sundays'
FROM (
SELECT SUM(amount) as am
FROM payment
WHERE WEEKDAY(DATE(payment_date)) = 6 
GROUP BY DATE(payment_date) 
) a


##### Approach : 
+ Here we have to calculate average daily revenue of all Sundays.
+ In the where condition `WEEKDAY()` function gives a numeric value which mapped to days and for Sunday it is 6.
+ And Rounding the Avg to 2 decimal places.

##### OUTPUT :
<img width="599" alt="Screenshot 2023-03-08 at 2 33 41 PM" src="https://user-images.githubusercontent.com/122472996/223669795-39c4a2b0-ec47-4be7-905c-023cfbfc0ee8.png">

### QUESTION 7:
Write a query to create a list that shows how much the average customer spent in total (customer life-time value) grouped by the different districts.

#### CODE:

WITH t1 AS (

SELECT DISTINCT a.district  , 
SUM(py.amount) OVER(PARTITION BY a.district )  AS sm 
FROM address a 
JOIN customer c
ON a.address_id = c.address_id 
JOIN payment py
ON py.customer_id = c.customer_id 
),
t2 AS (
SELECT a.district ,
COUNT(c.customer_id) AS cnt
FROM  address a 
JOIN customer c
ON a.address_id = c.address_id 
GROUP BY a.district
)
SELECT t1.district , t1.sm/t2.cnt AS 'average_amount'
FROM t1,t2
WHERE t1.district = t2.district


##### Approach: 
+ In this I have to CTEs as t1 and t2 .
+ t1 calculates the sum of amount per district
+ t2 calcultes the distinct district.


##### OUTPUT :
<img width="599" alt="Screenshot 2023-03-08 at 2 37 45 PM" src="https://user-images.githubusercontent.com/122472996/223670601-00c0a68a-74eb-464c-b5a0-777880711312.png">


### QUESTION 8:
Write a query to list down the highest overall revenue collected (sum of amount per title) by a film in each category. Result should display the film title, category name and total revenue.
eg.
+ "FOOL MOCKINGBIRD" "Action" 175.77
+ "DOGMA FAMILY" "Animation" 178.7
+ "BACKLASH UNDEFEATED" "Children" 158.81

#### CODE:

SELECT a.title , 
a.name , a.Total_Revenue
FROM (
SELECT f.title , 
ct.name , 
SUM(py.amount) as 'Total_Revenue' , 
DENSE_RANK() OVER(PARTITION BY ct.name ORDER BY sum(py.amount) DESC ) AS rn 
FROM film f 
JOIN film_category fc
ON f.film_id = fc.film_id
JOIN category ct
ON ct.category_id = fc.category_id
JOIN inventory inv
ON inv.film_id = f.film_id
JOIN rental r 
ON r.inventory_id = inv.inventory_id 
JOIN payment py 
ON py.rental_id = r.rental_id
GROUP BY  f.title , ct.name
) a
WHERE a.rn<=1

##### Approach :
+ Used the Window Function which partition by category name and order by the sum of the payment and giving them a dense rank.
+ And Putting condition where rank is less than equal to 1.

##### OUTPUT :
<img width="599" alt="Screenshot 2023-03-08 at 2 42 22 PM" src="https://user-images.githubusercontent.com/122472996/223671575-1962222d-58e7-4bc9-b87d-c006e10bee95.png">


### QUESTION 9:
Modify the table "rental" to be partitioned using PARTITION command based on ‘rental_date’ in below intervals:

` <2005
        between 2005–2010
	between 2011–2015
	between 2016–2020
	>2020 - Partitions are created yearly `

#### CODE:

ALTER TABLE rental 
PARTITION BY RANGE(YEAR(rental_date))
(
PARTITION rent_lt_2005 VALUES LESS THAN (2005),
PARTITION rent_between_2005_2010 VALUES LESS THAN (2011),
PARTITION rent_between_2011_2015 VALUES LESS THAN (2016),
PARTITION rent_between_2016_2020 VALUES LESS THAN (2021),
PARTITION rent_gt_2020 VALUES LESS THAN MAXVALUE
);


### QUESTION 10:
Modify the table "film" to be partitioned using PARTITION command based on ‘rating’ from below list. Further apply hash sub-partitioning based on ‘film_id’ into 4 sub-partitions.

#### CODE:


ALTER TABLE film
PARTITION BY LIST(rating)
SUBPARTITION BY HASH(film_id) SUBPARTITIONS 4
(
PARTITION PR values('R'),
PARTITION Pgs values('PG-13', 'PG'),
PARTITION GNC values('G', 'NC-17')
);



##### OUTPUT :

### QUESTION 11:
Write a query to count the total number of addresses from the “address” table where the ‘postal_code’ is of the below formats. Use regular expression.
+ 9*1*, 9*2, 9*3, 9*4, 9*5*

#### CODE:

SELECT count(postal_code) AS 'No_of_postal_code'
FROM address
WHERE postal_code REGEXP '^9[0-9][1-5][0-9]{2}$'



##### OUTPUT :
<img width="599" alt="Screenshot 2023-03-08 at 2 49 34 PM" src="https://user-images.githubusercontent.com/122472996/223673142-2dd52b0a-a629-4787-bdaf-6d8550267add.png">


### QUESTION 12:
Write a query to create a materialized view from the “payment” table where ‘amount’ is between(inclusive) $5 to $8. The view should manually refresh on demand. Also write a query to manually refresh the created materialized view.
#### CODE:

DELIMITER $$
CREATE EVENT refresh_payment_between_5_8      
ON SCHEDULE EVERY 1 DAY
DO
BEGIN
  CREATE OR REPLACE VIEW payment_between_5_8 AS
  SELECT *
  FROM payment
  WHERE amount BETWEEN 5 AND 8;
END$$
DELIMITER ;


SELECT * FROM payment_between_5_8;

##### OUTPUT :
<img width="570" alt="Screenshot 2023-03-08 at 2 51 35 PM" src="https://user-images.githubusercontent.com/122472996/223673596-2621ac76-20b0-4193-856a-89582d5c454a.png">


### QUESTION 13:
Write a query to list down the total sales of each staff with each customer from the ‘payment’ table. In the same result, list down the total sales of each staff i.e. sum of sales from all customers for a particular staff. Use the ROLLUP command. Also use GROUPING command to indicate null values.
#### CODE:

SELECT p.staff_id,p.customer_id,
GROUPING(p.staff_id) as staff,
GROUPING(p.customer_id) as customer,sum(p.amount) as sum_of_sales
FROM payment p
GROUP BY p.staff_id,p.customer_id
WITH ROLLUP;


##### OUTPUT :
<img width="570" alt="Screenshot 2023-03-08 at 2 53 21 PM" src="https://user-images.githubusercontent.com/122472996/223674036-f72fa252-35e1-456e-b0a2-e8d1176fea67.png">


### QUESTION 14:
Write a single query to display the customer_id, staff_id, payment_id, amount, amount on immediately previous payment_id, amount on immediately next payment_id ny_sales for the payments from customer_id ‘269’ to staff_id ‘1’.

#### CODE:

SELECT customer_id,payment_id,staff_id,
LEAD(amount) OVER(ORDER BY payment_id) next_payment,
LAG(amount) OVER(ORDER BY payment_id) previous_amount,
LEAD(amount) OVER(PARTITION BY customer_id,staff_id ORDER BY payment_id) AS next_payment_sales,
LAG(amount) OVER(PARTITION BY customer_id,staff_id ORDER BY payment_id) AS previous_payment_sales
FROM payment
WHERE customer_id=269 and staff_id=1;


##### OUTPUT :
<img width="784" alt="Screenshot 2023-03-08 at 2 58 42 PM" src="https://user-images.githubusercontent.com/122472996/223675237-c5273dd4-9fa9-45f6-bc02-bd877360bdaa.png">



## Atul's Approach



### QUESTION 1:
Write a query that gives an overview of how many films have replacements costs in the following cost ranges
low: 9.99 - 19.99
medium: 20.00 - 24.99
high: 25.00 - 29.99
#### CODE:

SELECT
SUM(
CASE
WHEN replacement_cost between 9.99 and 19.99  THEN 1 ELSE 0 END
)  AS 'no_of_low' ,
SUM(
CASE
WHEN replacement_cost between 20.00 and 24.99  THEN 1 ELSE 0 END
)  AS 'no_of_medium' ,
SUM(
CASE
WHEN replacement_cost between 25.00 and 29.99  THEN 1 ELSE 0 END
)  AS 'no_of_high'
FROM film

##### Approach : 
+ Here in this particular question we have been asked to give the overview of the films where the replacement_cost is in the given range categorizing it into low,medium and high
+ Logic in the case statement is if the particular replacement_cost is within the specified range for the particular category then returning 1 else 0 which we are summing up using SUM() function which results in total number of films in that particular range


##### OUTPUT :

<img width="441" alt="Screenshot 2023-03-08 at 2 11 38 PM" src="https://user-images.githubusercontent.com/122472996/223664760-6613db6b-21d0-4c2c-80ee-180e3cbf0f4c.png">

### QUESTION 2:
Write a query to create a list of the film titles including their film title, film length and film category name ordered descendingly by the film length. Filter the results to only the movies in the category 'Drama' or 'Sports'.
+ "STAR OPERATION" "Sports" 181
+ "JACKET FRISCO" "Drama" 181
#### CODE:

SELECT title AS 'Film Title' , 
c.name AS 'Film Category', 
length_ AS 'Film Length' 
FROM film f JOIN film_category fc
ON f.film_id = fc.film_id
JOIN category c 
ON fc.category_id = c.category_id
WHERE c.name = 'Drama' or c.name='Sports' 
ORDER BY f.length_ DESC ;

##### Approach :  
 + Here we have to create a list of the film titles including their film title, film length and film category name ordered descendingly by the film length for the category “sports” and “drama”.
+ So for that I have joined three tables i.e., film, film_category, category and extracted the film_title, film_length and film_category by applying the filter for category name to be only “Sports” and “Drama” ordering by the length of the film in descending order.


##### OUTPUT : 
<img width="585" alt="Screenshot 2023-03-08 at 2 16 07 PM" src="https://user-images.githubusercontent.com/122472996/223665771-b6aac043-abe7-47ff-b497-e923908993e9.png">

### QUESTION 3:
Write a query to create a list of the addresses that are not associated to any customer.

#### CODE:

SELECT a.address_id ,
a.address
FROM customer c 
RIGHT JOIN address a
ON c.address_id = a.address_id
WHERE c.customer_id IS NULL

##### Approach :
+ We have been asked to create a list of the addresses that are not associated to any customer
+ So For that I have used two tables i.e., `address` and `customer`.
+ I have used right join above to retrieve all the data from the address table irrespective of whether it is present in the customer table. So whichever customer_id is not associated with any address will result in null values which i am using in the where clause to filter out the data where customer_id is null which will give me all the address which are not related to any customer.


##### OUTPUT : 
<img width="585" alt="Screenshot 2023-03-08 at 2 19 26 PM" src="https://user-images.githubusercontent.com/122472996/223666541-33b758d1-0e0a-4340-bcb3-2994a7e688e8.png">


### QUESTION 4:
Write a query to create a list of the revenue (sum of amount) grouped by a column in the format "country, city" ordered in decreasing amount of revenue.
* eg. "Poland, Bydgoszcz" 52.88

#### CODE:

SELECT concat(cty.country,' , ' ,ct.city  ) AS country_city ,
SUM(py.amount)  AS 'List of the revenue'
FROM rental r 
JOIN customer c
ON r.customer_id = c.customer_id
JOIN address a 
ON c.address_id = a.address_id
JOIN city ct 
ON a.city_id = ct.city_id
JOIN country cty
ON cty.country_id = ct.country_id
JOIN payment py 
ON r.rental_id = py.rental_id
GROUP BY cty.country, ct.city 

##### Approach: 
+ Here we need to create a list of the revenue based on country, city according to decreasing amount of revenue
+ So for getting the desired result I have joined 4 tables i.e., country, city, address and payment to fetch country name, city name which I am concatenating using CONCAT() function
+ Also calculated the revenue by using window function which is calculating the sum of amount partitioning by Country,City and then ordering revenue by descending order
##### OUTPUT :
<img width="599" alt="Screenshot 2023-03-08 at 2 24 02 PM" src="https://user-images.githubusercontent.com/122472996/223667564-13bd53f8-cb88-4ee6-8c42-b0ae74aee6bc.png">

### QUESTION 5:
Write a query to create a list with the average of the sales amount each staff_id has per customer.
result: 
+ 2 56.64
+ 1 55.91

#### CODE:

SELECT staff_id, ROUND(AVG(sum_amount), 2) AS sales_amount
FROM (
SELECT DISTINCT staff_id, customer_id, 
SUM(amount) OVER(PARTITION BY staff_id,customer_id) AS sum_amount 
FROM payment 
)a
GROUP BY staff_id

##### Approach :
+ Here we are asked to create a list with the average of sales per customer with each staff id.
+ In this I have used the window function to calculate the sum_amount 
##### OUTPUT :
<img width="599" alt="Screenshot 2023-03-08 at 2 26 51 PM" src="https://user-images.githubusercontent.com/122472996/223668179-13554356-959b-4726-bcce-986f70cbb1e0.png">

### QUESTION 6:
Write a query that shows average daily revenue of all Sundays.

#### CODE:

SELECT ROUND(avg(am),2) AS 'average daily revenue of all sundays'
FROM (
SELECT SUM(amount) as am
FROM payment
WHERE WEEKDAY(DATE(payment_date)) = 6 
GROUP BY DATE(payment_date) 
) a


##### Approach : 
+ Here we have to calculate average daily revenue of all Sundays.
+ In the where condition `WEEKDAY()` function gives a numeric value which mapped to days and for Sunday it is 6.
+ And Rounding the Avg to 2 decimal places.

##### OUTPUT :
<img width="599" alt="Screenshot 2023-03-08 at 2 33 41 PM" src="https://user-images.githubusercontent.com/122472996/223669795-39c4a2b0-ec47-4be7-905c-023cfbfc0ee8.png">

### QUESTION 7:
Write a query to create a list that shows how much the average customer spent in total (customer life-time value) grouped by the different districts.

#### CODE:

WITH t1 AS (

SELECT DISTINCT a.district  , 
SUM(py.amount) OVER(PARTITION BY a.district )  AS sm 
FROM address a 
JOIN customer c
ON a.address_id = c.address_id 
JOIN payment py
ON py.customer_id = c.customer_id 
),
t2 AS (
SELECT a.district ,
COUNT(c.customer_id) AS cnt
FROM  address a 
JOIN customer c
ON a.address_id = c.address_id 
GROUP BY a.district
)
SELECT t1.district , t1.sm/t2.cnt AS 'average_amount'
FROM t1,t2
WHERE t1.district = t2.district


##### Approach: 
+ In this I have to CTEs as t1 and t2 .
+ t1 calculates the sum of amount per district
+ t2 calcultes the distinct district.


##### OUTPUT :
<img width="599" alt="Screenshot 2023-03-08 at 2 37 45 PM" src="https://user-images.githubusercontent.com/122472996/223670601-00c0a68a-74eb-464c-b5a0-777880711312.png">


### QUESTION 8:
Write a query to list down the highest overall revenue collected (sum of amount per title) by a film in each category. Result should display the film title, category name and total revenue.
eg.
+ "FOOL MOCKINGBIRD" "Action" 175.77
+ "DOGMA FAMILY" "Animation" 178.7
+ "BACKLASH UNDEFEATED" "Children" 158.81

#### CODE:

SELECT a.title , 
a.name , a.Total_Revenue
FROM (
SELECT f.title , 
ct.name , 
SUM(py.amount) as 'Total_Revenue' , 
DENSE_RANK() OVER(PARTITION BY ct.name ORDER BY sum(py.amount) DESC ) AS rn 
FROM film f 
JOIN film_category fc
ON f.film_id = fc.film_id
JOIN category ct
ON ct.category_id = fc.category_id
JOIN inventory inv
ON inv.film_id = f.film_id
JOIN rental r 
ON r.inventory_id = inv.inventory_id 
JOIN payment py 
ON py.rental_id = r.rental_id
GROUP BY  f.title , ct.name
) a
WHERE a.rn<=1

##### Approach :
+ Used the Window Function which partition by category name and order by the sum of the payment and giving them a dense rank.
+ And Putting condition where rank is less than equal to 1.

##### OUTPUT :
<img width="599" alt="Screenshot 2023-03-08 at 2 42 22 PM" src="https://user-images.githubusercontent.com/122472996/223671575-1962222d-58e7-4bc9-b87d-c006e10bee95.png">


### QUESTION 9:
Modify the table "rental" to be partitioned using PARTITION command based on ‘rental_date’ in below intervals:

` <2005
        between 2005–2010
	between 2011–2015
	between 2016–2020
	>2020 - Partitions are created yearly `

#### CODE:

ALTER TABLE rental 
PARTITION BY RANGE(YEAR(rental_date))
(
PARTITION rent_lt_2005 VALUES LESS THAN (2005),
PARTITION rent_between_2005_2010 VALUES LESS THAN (2011),
PARTITION rent_between_2011_2015 VALUES LESS THAN (2016),
PARTITION rent_between_2016_2020 VALUES LESS THAN (2021),
PARTITION rent_gt_2020 VALUES LESS THAN MAXVALUE
);


### QUESTION 10:
Modify the table "film" to be partitioned using PARTITION command based on ‘rating’ from below list. Further apply hash sub-partitioning based on ‘film_id’ into 4 sub-partitions.

#### CODE:


ALTER TABLE film
PARTITION BY LIST(rating)
SUBPARTITION BY HASH(film_id) SUBPARTITIONS 4
(
PARTITION PR values('R'),
PARTITION Pgs values('PG-13', 'PG'),
PARTITION GNC values('G', 'NC-17')
);



##### OUTPUT :

### QUESTION 11:
Write a query to count the total number of addresses from the “address” table where the ‘postal_code’ is of the below formats. Use regular expression.
+ 9*1*, 9*2, 9*3, 9*4, 9*5*

#### CODE:

SELECT count(postal_code) AS 'No_of_postal_code'
FROM address
WHERE postal_code REGEXP '^9[0-9][1-5][0-9]{2}$'



##### OUTPUT :
<img width="599" alt="Screenshot 2023-03-08 at 2 49 34 PM" src="https://user-images.githubusercontent.com/122472996/223673142-2dd52b0a-a629-4787-bdaf-6d8550267add.png">


### QUESTION 12:
Write a query to create a materialized view from the “payment” table where ‘amount’ is between(inclusive) $5 to $8. The view should manually refresh on demand. Also write a query to manually refresh the created materialized view.
#### CODE:

DELIMITER $$
CREATE EVENT refresh_payment_between_5_8      
ON SCHEDULE EVERY 1 DAY
DO
BEGIN
  CREATE OR REPLACE VIEW payment_between_5_8 AS
  SELECT *
  FROM payment
  WHERE amount BETWEEN 5 AND 8;
END$$
DELIMITER ;


SELECT * FROM payment_between_5_8;

##### OUTPUT :
<img width="570" alt="Screenshot 2023-03-08 at 2 51 35 PM" src="https://user-images.githubusercontent.com/122472996/223673596-2621ac76-20b0-4193-856a-89582d5c454a.png">


### QUESTION 13:
Write a query to list down the total sales of each staff with each customer from the ‘payment’ table. In the same result, list down the total sales of each staff i.e. sum of sales from all customers for a particular staff. Use the ROLLUP command. Also use GROUPING command to indicate null values.
#### CODE:

SELECT p.staff_id,p.customer_id,
GROUPING(p.staff_id) as staff,
GROUPING(p.customer_id) as customer,sum(p.amount) as sum_of_sales
FROM payment p
GROUP BY p.staff_id,p.customer_id
WITH ROLLUP;


##### OUTPUT :
<img width="570" alt="Screenshot 2023-03-08 at 2 53 21 PM" src="https://user-images.githubusercontent.com/122472996/223674036-f72fa252-35e1-456e-b0a2-e8d1176fea67.png">


### QUESTION 14:
Write a single query to display the customer_id, staff_id, payment_id, amount, amount on immediately previous payment_id, amount on immediately next payment_id ny_sales for the payments from customer_id ‘269’ to staff_id ‘1’.

#### CODE:

SELECT customer_id,payment_id,staff_id,
LEAD(amount) OVER(ORDER BY payment_id) next_payment,
LAG(amount) OVER(ORDER BY payment_id) previous_amount,
LEAD(amount) OVER(PARTITION BY customer_id,staff_id ORDER BY payment_id) AS next_payment_sales,
LAG(amount) OVER(PARTITION BY customer_id,staff_id ORDER BY payment_id) AS previous_payment_sales
FROM payment
WHERE customer_id=269 and staff_id=1;


##### OUTPUT :
<img width="784" alt="Screenshot 2023-03-08 at 2 58 42 PM" src="https://user-images.githubusercontent.com/122472996/223675237-c5273dd4-9fa9-45f6-bc02-bd877360bdaa.png">
