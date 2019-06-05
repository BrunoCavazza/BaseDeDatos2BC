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
