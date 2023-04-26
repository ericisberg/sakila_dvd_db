/* Query 1 - Which customers have rented the most movies in each country, and how many movies have they each rented? */

/* Uses Aliases, Joins, CTE, and Aggregations (COUNT) */
WITH customer_country_rentals AS (
  SELECT 
    c.customer_id, 
    co.country, 
    COUNT(*) AS num_rentals 
  FROM 
    customer c 
    JOIN rental r ON c.customer_id = r.customer_id 
    JOIN address a ON c.address_id = a.address_id 
    JOIN city ci ON a.city_id = ci.city_id 
    JOIN country co ON ci.country_id = co.country_id 
  GROUP BY 
    c.customer_id, 
    co.country
) 
SELECT 
  ccr.country, 
  c.first_name, 
  c.last_name, 
  ccr.num_rentals 
FROM 
  customer c 
  JOIN customer_country_rentals ccr ON c.customer_id = ccr.customer_id 
WHERE 
  (ccr.country, ccr.num_rentals) IN (
    SELECT 
      country, 
      MAX(num_rentals) 
    FROM 
      customer_country_rentals 
    GROUP BY 
      country
  ) 
ORDER BY 
  ccr.country,
  ccr.num_rentals DESC;

/* Query 2 - Which customers have generated the highest total revenue from DVD rentals, and what is their total revenue? */

/* Uses Aliases, Joins, and Aggregations (SUM) */
SELECT 
  c.customer_id, 
  c.first_name || ' ' || c.last_name AS customer_name, 
  SUM(p.amount) AS total_revenue 
FROM 
  customer c 
  JOIN payment p ON c.customer_id = p.customer_id 
GROUP BY 
  c.customer_id 
ORDER BY 
  total_revenue DESC;
	
/* Query 3 - What is the city with the highest average number of rentals per customer? */

/* Uses Aliases, Joins, CTE, and Aggregations (COUNT), (AVG) */
WITH rentals_per_customer AS (
  SELECT 
    c.customer_id, 
    COUNT(*) AS num_rentals 
  FROM 
    customer c 
    JOIN rental r ON c.customer_id = r.customer_id 
  GROUP BY 
    c.customer_id
) 
SELECT 
  ci.city, 
  co.country, 
  AVG(rpc.num_rentals) AS avg_rentals_per_customer 
FROM 
  rentals_per_customer rpc 
  JOIN address a ON rpc.customer_id = a.address_id 
  JOIN city ci ON a.city_id = ci.city_id 
  JOIN country co ON ci.country_id = co.country_id 
GROUP BY 
  ci.city, 
  co.country 
ORDER BY 
  avg_rentals_per_customer DESC, 
  co.country;


/* Query 4 - What is the average rental duration for each distinct film category, and how does it compare to the overall average rental duration for all categories?  */

/* Uses Aliases, Joins, CTE, and Aggregations(AVG), and Windows Functions(2 using OVER) */
WITH category_avg_duration AS (
  SELECT 
    c.category_id, 
    AVG(f.rental_duration) OVER (PARTITION BY c.category_id) AS avg_rental_duration, 
    AVG(f.rental_duration) OVER () AS overall_avg_rental_duration 
  FROM 
    category c 
    JOIN film_category fc ON c.category_id = fc.category_id 
    JOIN film f ON fc.film_id = f.film_id
) 
SELECT 
  DISTINCT ON (c.name) c.name AS category_name, 
  cad.avg_rental_duration, 
  cad.overall_avg_rental_duration 
FROM 
  category_avg_duration cad 
  JOIN category c ON cad.category_id = c.category_id 
ORDER BY 
  c.name;