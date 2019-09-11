-- ---------------------------------------------------------------------------- 
Clase 9:
-- ---------------------------------------------------------------------------- 
-- 1
SELECT country.country, COUNT(*) 
	FROM city INNER JOIN country USING (country_id) 
	GROUP BY city.country_id 
	ORDER BY country.country, country.country_id

-- ---------------------------------------------------------------------------- 
-- 2
SELECT country.country, COUNT(*) 
	FROM city INNER JOIN country USING (country_id) 
	GROUP BY city.country_id HAVING count(*) > 10 
	ORDER BY count(*) DESC
	
-- ---------------------------------------------------------------------------- 
-- 3
SELECT c.customer_id, c.last_name, c.first_name, 
	(SELECT count(*) FROM rental r1 WHERE r1.customer_id = c.customer_id) AS "Total Films",
	(SELECT SUM(amount)FROM payment p1 WHERE p1.customer_id = c.customer_id
	GROUP BY customer_id) AS "TotalMoney" 
	FROM customer c ORDER BY TotalMoney DESC

-- ---------------------------------------------------------------------------- 
-- 4
SELECT category.name, AVG(film.`length`) AS "average"
	FROM film INNER JOIN film_category 
	USING (film_id) INNER JOIN category 
	USING (category_id) GROUP BY category.category_id
	HAVING AVG(`length`) > (SELECT AVG(`length`) FROM film)
	ORDER BY AVG(film.`length`) DESC
  
-- ---------------------------------------------------------------------------- 	
-- 5
SELECT film.rating, SUM(payment.amount) AS "sales" FROM film 
	INNER JOIN inventory USING (film_id) 
	INNER JOIN rental USING (inventory_id)
	INNER JOIN payment USING (rental_id)
	GROUP BY film.rating
	ORDER BY sales DESC
-- ---------------------------------------------------------------------------- 
Clase 11:
-- ---------------------------------------------------------------------------- 
-- 1. 
SELECT film.title 
FROM film 
WHERE film.film_id NOT IN (SELECT film_id 
FROM inventory); 

-- ---------------------------------------------------------------------------- 
-- 2. 
SELECT film.title,inventory_id,rental.rental_id 
FROM film 
INNER JOIN inventory USING (film_id) 
LEFT OUTER JOIN rental USING (inventory_id) 
WHERE rental.rental_id IS NULL; 

-- ----------------------------------------------------------------------------
-- 3
SELECT customer.first_name,customer.last_name,inventory.store_id,film.title, 
rental.rental_date,rental.return_date 
FROM film 
INNER JOIN inventory USING (film_id) 
INNER JOIN rental USING (inventory_id) 
INNER JOIN customer USING (customer_id) 
WHERE rental.return_date IS NOT NULL 
ORDER BY inventory.store_id,customer.last_name; 


-- ----------------------------------------------------------------------------
-- 4
SELECT CONCAT(c.city, ', ', co.country) AS store,
CONCAT(m.first_name, ' ', m.last_name) AS manager,
SUM(p.amount) AS total_sales
FROM payment AS p
INNER JOIN rental AS r ON p.rental_id = r.rental_id
INNER JOIN inventory AS i ON r.inventory_id = i.inventory_id
INNER JOIN store AS s ON i.store_id = s.store_id
INNER JOIN address AS a ON s.address_id = a.address_id
INNER JOIN city AS c ON a.city_id = c.city_id
INNER JOIN country AS co ON c.country_id = co.country_id
INNER JOIN staff AS m ON s.manager_staff_id = m.staff_id
GROUP BY s.store_id
ORDER BY co.country, c.city;

-- ----------------------------------------------------------------------------
-- 5
SELECT actor.actor_id, 
actor.first_name, 
actor.last_name, 
COUNT(actor_id) AS film_count 
FROM actor 
INNER JOIN film_actor USING (actor_id) 
GROUP BY actor_id, actor.first_name, actor.last_name 
HAVING COUNT(actor_id) >= (SELECT COUNT(film_id) 
FROM film_actor 
GROUP BY actor_id 
ORDER BY 1 DESC 
LIMIT 1) 
ORDER BY film_count DESC

-- ---------------------------------------------------------------------------- 
Clase 13:
-- ---------------------------------------------------------------------------- 
-- 1
INSERT INTO sakila.customer
(store_id, first_name, last_name, email, address_id, active)
SELECT 1, 'Pepe', 'Suarez', 'pepesuarez@gmail.com', MAX(a.address_id), 1
FROM address a
WHERE (SELECT c.country_id
        FROM country c, city c1
        WHERE c.country = "United States"
        AND c.country_id = c1.country_id
        AND c1.city_id = a.city_id);


SELECT *
FROM customer
WHERE last_name = "Suarez";

-- ---------------------------------------------------------------------------- 
-- 2
INSERT INTO sakila.rental
(rental_date, inventory_id, customer_id, return_date, staff_id)
SELECT CURRENT_TIMESTAMP, 
        (SELECT MAX(r.inventory_id)
         FROM inventory r
         INNER JOIN film USING(film_id)
         WHERE film.title = "ARABIA DOGMA" 
         LIMIT 1), 
         600, 
         NULL,
         (SELECT staff_id
          FROM staff
          INNER JOIN store USING(store_id)
          WHERE store.store_id = 2
          LIMIT 1);
-- ---------------------------------------------------------------------------- 
-- 3
UPDATE sakila.film
SET release_year='2001'
WHERE rating = "G";

UPDATE sakila.film
SET release_year='2005'
WHERE rating = "PG";

UPDATE sakila.film
SET release_year='2010'
WHERE rating = "PG-13";

UPDATE sakila.film
SET release_year='2015'
WHERE rating = "R";

UPDATE sakila.film
SET release_year='2020'
WHERE rating = "NC-17";
-- ---------------------------------------------------------------------------- 
-- 4
SELECT rental_id, rental_rate, customer_id, staff_id
FROM film
INNER JOIN inventory USING(film_id)
INNER JOIN rental USING(inventory_id)
WHERE rental.return_date IS NULL
LIMIT 1;

UPDATE sakila.rental
SET  return_date=CURRENT_TIMESTAMP
WHERE rental_id=11496;
-- ---------------------------------------------------------------------------- 
-- 5 
DELETE FROM payment
 WHERE rental_id IN (SELECT rental_id 
                       FROM rental
                      INNER JOIN inventory USING (inventory_id) 
                      WHERE film_id = 1);

DELETE FROM rental
 WHERE inventory_id IN (SELECT inventory_id 
                         FROM inventory
                        WHERE film_id = 1);

DELETE FROM inventory WHERE film_id = 1;

DELETE film_actor FROM film_actor WHERE film_id = 1;

DELETE film_category FROM film_category WHERE film_id = 1;

DELETE film FROM film WHERE film_id = 1;
-- ---------------------------------------------------------------------------- 
-- 6
SELECT inventory_id, film_id

FROM inventory

WHERE inventory_id NOT IN (SELECT inventory_id

FROM inventory

    INNER JOIN rental USING (inventory_id)

    WHERE return_date IS NULL)

# inventory id to use: 10

# film id to use: 2

INSERT INTO sakila.rental

(rental_date, inventory_id, customer_id, staff_id)

VALUES(

CURRENT_DATE(),

10,

(SELECT customer_id FROM customer ORDER BY customer_id DESC LIMIT 1),

(SELECT staff_id FROM staff WHERE store_id = (SELECT store_id FROM inventory WHERE inventory_id = 10))

);

INSERT INTO sakila.payment

(customer_id, staff_id, rental_id, amount, payment_date)

VALUES(

(SELECT customer_id FROM customer ORDER BY customer_id DESC LIMIT 1),

(SELECT staff_id FROM staff LIMIT 1),

(SELECT rental_id FROM rental ORDER BY rental_id DESC LIMIT 1) ,

(SELECT rental_rate FROM film WHERE film_id = 2),

CURRENT_DATE());

-- ---------------------------------------------------------------------------- 
Clase 14
-- ---------------------------------------------------------------------------- 
-- 1
SELECT CONCAT_WS(" ",first_name,last_name) as full_name, address.address, city.city
FROM customer 
    INNER JOIN address USING(address_id)
    INNER JOIN city USING(city_id)
    INNER JOIN country USING(country_id)
WHERE country.country LIKE 'Argentina';	
-- ---------------------------------------------------------------------------- 
-- 2
SELECT title,
`language`.name, 
CASE
    WHEN rating = 'G' THEN 'All Ages Are Admitted.'
    WHEN rating = 'PG' THEN 'Some Material May Not Be Suitable For Children.'
    WHEN rating = 'PG-13' THEN 'Some Material May Be Inappropriate For Children Under 13.'
    WHEN rating = 'R' THEN 'Under 17 Requires Accompanying Parent Or Adult Guardian.'
    WHEN rating = 'NC-17' THEN 'No One 17 and Under Admitted.'
END AS rating_description
  FROM film
    INNER JOIN `language` USING (language_id);
-- ---------------------------------------------------------------------------- 
-- 3
SELECT title, release_year
  FROM film 
    INNER JOIN film_actor USING(film_id)
    INNER JOIN actor USING(actor_id)
WHERE CONCAT_WS(" ",first_name,last_name) LIKE TRIM(UPPER("   johNNy lollobRigidA     "));
-- ---------------------------------------------------------------------------- 
-- 4
SELECT film.title,
       CONCAT_WS(" ", customer.first_name, customer.last_name) as full_name,
       CASE WHEN rental.return_date IS NOT NULL THEN 'Yes'
       ELSE 'No' END AS was_returned,
       MONTHNAME(rental.rental_date) as month
  FROM film
    INNER JOIN inventory USING(film_id)
    INNER JOIN rental USING(inventory_id)
    INNER JOIN customer USING(customer_id)
WHERE MONTHNAME(rental.rental_date) LIKE 'May'
   OR MONTHNAME(rental.rental_date) LIKE 'June';
-- ---------------------------------------------------------------------------- 
-- 5
SELECT CAST(last_update AS DATE) as only_date
FROM rental;

SELECT CONVERT("2006-02-15", DATETIME);
-- ---------------------------------------------------------------------------- 
-- 6
SELECT rental_id, IFNULL(return_date, 'La pelicula no fue devuelta aun') as fecha_de_devolucion
  FROM rental
WHERE rental_id = 12610
  OR rental_id = 12611;

SELECT rental_id, ISNULL(return_date) as pelicula_faltante
  FROM rental
WHERE rental_id = 12610
  OR rental_id = 12611;
  
SELECT COALESCE(NULL,
                NULL,
                (SELECT return_date
                FROM rental
                WHERE rental_id = 12610), -- null date
                (SELECT return_date
                FROM rental
                WHERE rental_id = 12611)) as primer_valor_no_nulo;
-- ---------------------------------------------------------------------------- 
Clase 15
-- ---------------------------------------------------------------------------- 
-- 1
CREATE OR REPLACE VIEW list_of_customers AS
    SELECT customer_id,
        CONCAT_WS(" " ,first_name, last_name) as full_name,
        address,
        postal_code,
        phone,
        city,
        country,
        CASE 
            WHEN active = 1 THEN 'active' 
            ELSE 'inactive' 
        END AS status,
        store_id
    FROM customer
        INNER JOIN address USING(address_id)
        INNER JOIN city USING(city_id)
        INNER JOIN country USING(country_id);

SELECT * FROM list_of_customers;
-- ---------------------------------------------------------------------------- 
-- 2
CREATE OR REPLACE VIEW film_details AS
    SELECT film_id,
        title,
        description,
        name,
        rental_rate,
        `length`,
        rating,
        GROUP_CONCAT(DISTINCT CONCAT_WS(" " ,first_name, last_name) SEPARATOR ',') AS actores
    FROM film
        INNER JOIN film_category USING(film_id)
        INNER JOIN category USING(category_id)
        INNER JOIN film_actor USING(film_id)
        INNER JOIN actor USING(actor_id)
    GROUP BY 1,4;

SELECT * FROM film_details;
-- ---------------------------------------------------------------------------- 
-- 3
CREATE OR REPLACE VIEW sales_by_film_category AS
    SELECT DISTINCT category.name, SUM(amount) as total_rental FROM category
                    INNER JOIN film_category USING(category_id)
                    INNER JOIN film USING(film_id)
                    INNER JOIN inventory USING(film_id)
                    INNER JOIN rental USING(inventory_id)
                    INNER JOIN payment USING(rental_id)
                GROUP BY 1;


SELECT * FROM sales_by_film_category;
-- ---------------------------------------------------------------------------- 
-- 4
CREATE OR REPLACE VIEW actor_information AS
    SELECT actor_id as id,
        first_name,
        last_name,
        (SELECT COUNT(film_id)
        FROM film
            INNER JOIN film_actor USING(film_id)
            INNER JOIN actor USING(actor_id)
        WHERE actor_id = id) AS films_participated_in
      FROM actor;

SELECT * FROM actor_information;
-- ---------------------------------------------------------------------------- 
Clase 16
-- ----------------------------------------------------------------------------
CREATE TABLE `employees` (
  `employeeNumber` int(11) NOT NULL,
  `lastName` varchar(50) NOT NULL,
  `firstName` varchar(50) NOT NULL,
  `extension` varchar(10) NOT NULL,
  `email` varchar(100) NOT NULL,
  `officeCode` varchar(10) NOT NULL,
  `reportsTo` int(11) DEFAULT NULL,
  `jobTitle` varchar(50) NOT NULL,
  PRIMARY KEY (`employeeNumber`)
);

INSERT INTO `employees`(`employeeNumber`,`lastName`,`firstName`,`extension`,`email`,`officeCode`,`reportsTo`,`jobTitle`) VALUES 
(1002,'Murphy','Diane','x5800','dmurphy@classicmodelcars.com','1',NULL,'President'),
(1056,'Patterson','Mary','x4611','mpatterso@classicmodelcars.com','1',1002,'VP Sales'),
(1076,'Firrelli','Jeff','x9273','jfirrelli@classicmodelcars.com','1',1002,'VP Marketing');

CREATE TABLE employees_audit (
    id INT AUTO_INCREMENT PRIMARY KEY,
    employeeNumber INT NOT NULL,
    lastname VARCHAR(50) NOT NULL,
    changedat DATETIME DEFAULT NULL,
    action VARCHAR(50) DEFAULT NULL
);
-- ---------------------------------------------------------------------------- 
-- 1
INSERT INTO `employees`(`employeeNumber`,`lastName`,`firstName`,`extension`,`email`,`officeCode`,`reportsTo`,`jobTitle`) VALUES 
(1056,'Patterson','Mary','x4611',NULL,'1',1002,'VP Sales');
-- ---------------------------------------------------------------------------- 
-- 2
UPDATE employees set employeeNumber = employeeNumber - 20;

UPDATE employees set employeeNumber = employeeNumber + 20;
-- ---------------------------------------------------------------------------- 
-- 3

ALTER TABLE employees
ADD age TINYINT UNSIGNED DEFAULT 69;

ALTER TABLE employees
   ADD CONSTRAINT age CHECK(age >= 16 AND age <= 70);
-- ---------------------------------------------------------------------------- 
-- 5

ALTER TABLE employees
    ADD COLUMN lastUpdate DATETIME;


ALTER TABLE employees
    ADD COLUMN lastUpdateUser VARCHAR(255); 


CREATE TRIGGER before_employee_update 
    BEFORE UPDATE ON employees
    FOR EACH ROW 
BEGIN
     SET NEW.lastUpdate=NOW();
     SET NEW.lastUpdateUser=CURRENT_USER;
END;


update employees set lastName = 'Phanny' where employeeNumber = 1076;
-- ---------------------------------------------------------------------------- 
-- 6
BEGIN
    INSERT INTO film_text (film_id, title, description)
        VALUES (new.film_id, new.title, new.description);
END

BEGIN
    IF (old.title != new.title) OR (old.description != new.description) OR (old.film_id != new.film_id)
    THEN
        UPDATE film_text
            SET title=new.title,
                description=new.description,
                film_id=new.film_id
        WHERE film_id=old.film_id;
    END IF;
END

BEGIN
    DELETE FROM film_text WHERE film_id = old.film_id;
END
-- ---------------------------------------------------------------------------- 
