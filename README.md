# Sakila DVD Rental Database Analysis

This GitHub project aims to provide a comprehensive analysis of the Sakila DVD Rental database, which contains information about a company that rents movie DVDs. By leveraging SQL queries, this project explores various aspects of the customer base, including movie-watching patterns across different customer groups, payment earnings comparisons, and store performance evaluations.
## Project Goals
- Identify the most popular family-friendly movie genres among Sakila DVD Rental customers.
- Determine the minimum and maximum rental durations for family-friendly movies, along with percentile values.
- Find the maximum rental count between 2005 and 2006 for Store 1 and Store 2.
- Determine the top 5 most common cities that Sakila DVD Rental customers are from.


## Data Schema

The project utilizes the following schema for the Sakila DVD Rental database:

- `film`: Contains information about individual films, such as their titles, genres, and rental durations.
- `category`: Stores the movie categories.
- `film_category`: Connects films with their corresponding categories.
- `inventory`: Holds information about the DVD copies available for rental.
- `rental`: Contains rental details, including rental dates and inventory associations.
- `customer`: Stores customer information.
- `address`: Holds address information for customers.
- `city`: Contains city data, including city names.

## SQL Queries Used### SQL Queries Used
To achieve the project goals, the following SQL queries were utilized:

Query 1: Most Popular Family-Friendly Movie Genres
This query retrieves the count of rentals for each family-friendly movie genre and sorts the results in ascending order by genre name.
   ```sql
   SELECT c.name AS category_name, COUNT(r.rental_id) AS rental_count
   FROM film f
   JOIN film_category fc ON f.film_id = fc.film_id
   JOIN category c ON fc.category_id = c.category_id
   JOIN inventory i ON f.film_id = i.film_id
   JOIN rental r ON i.inventory_id = r.inventory_id
   WHERE c.name IN ('Animation', 'Children', 'Classics', 'Comedy', 'Family', 'Music')
   GROUP BY c.name
   ORDER BY c.name ASC;
```

Query 2: Rental Durations and Percentiles for Family-Friendly Movies
This query calculates the minimum, maximum, and average rental durations for each family-friendly movie genre. It also assigns a percentile value to each genre based on the average rental duration.
   ```sql

WITH category_rental_duration AS (
    SELECT category.name, 
        MIN(film.rental_duration) AS min_rental_duration, 
        MAX(film.rental_duration) AS max_rental_duration, 
        AVG(film.rental_duration) AS avg_rental_duration
    FROM category 
    JOIN film_category ON category.category_id = film_category.category_id 
    JOIN film ON film_category.film_id = film.film_id 
    WHERE category.name IN ('Animation', 'Children', 'Classics', 'Comedy', 'Family', 'Music')
    GROUP BY category.name
)
SELECT name, min_rental_duration, max_rental_duration, 
    NTILE(4) OVER (ORDER BY avg_rental_duration) AS percentile
FROM category_rental_duration;
```
  
Query 3: Maximum Rental Count for Stores 1 and 2 (2005-2006)
This query determines the maximum rental count for each store (Store 1 and Store 2) between the years 2005 and 2006. The results are grouped by year and store, sorted in descending order by year and maximum rental count.
  ```sql
SELECT year, store_id, max(rental_count) as max_rental_count
FROM (
  SELECT EXTRACT(YEAR FROM r.rental_date) AS year, i.store_id, COUNT(*) AS rental_count
  FROM rental r
  JOIN inventory i ON r.inventory_id = i.inventory_id
  GROUP BY year, i.store_id
) as rental_counts
GROUP BY year, store_id
ORDER BY year DESC, max_rental_count DESC;
```
Query 4: Top 5 Most Common Customer Cities
This query identifies the top 5 cities with the highest number of customers. It counts the number of customers associated with each city, ranks them in descending order by count, and selects the top 5 cities based on their ranks.
  ```sql
WITH city_counts AS (
  SELECT city.city, COUNT(*) AS count
  FROM customer
  JOIN address ON customer.address_id = address.address_id
  JOIN city ON address.city_id = city.city_id
  GROUP BY city.city
), 
ranked_cities AS (
  SELECT city, count, ROW_NUMBER() OVER (ORDER BY count DESC) AS rank
  FROM city_counts
)
SELECT city, count
FROM ranked_cities
WHERE rank <= 5;
```
