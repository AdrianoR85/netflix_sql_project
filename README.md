# Netflix Data Analysis with PostgreSQL

## About the Project

This project consists of a data analysis of the Netflix catalog using PostgreSQL. The objective is to solve 15 specific business problems through SQL queries that explore different aspects of the content available on the platform.

## Schemas of Netflix
```sql
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
	show_id	VARCHAR(5),
	type    VARCHAR(10),
	title	VARCHAR(250),
	director VARCHAR(550),
	casts	VARCHAR(1050),
	country	VARCHAR(550),
	date_added	VARCHAR(55),
	release_year	INT,
	rating	VARCHAR(15),
	duration	VARCHAR(15),
	listed_in	VARCHAR(250),
	description VARCHAR(550)
);
SELECT * FROM netflix;
```
## Business Problems Solved

1. **Count the number of Movies vs TV Shows**
   ```sql
   SELECT
       type,
       COUNT(*) AS total_by_type
   FROM netflix
   GROUP BY 1;
   ```

2. **Find the most common rating for movies and TV shows**
   ```sql
   SELECT 
       type,
       rating,
       total
   FROM
   (SELECT 
       type,
       rating,
       COUNT(*) as total,
       RANK() OVER (PARTITION BY type ORDER BY COUNT(*) DESC) AS raking
   FROM netflix
   GROUP BY 1,2
   ) AS t WHERE raking = 1;
   ```

3. **List all movies released in a specific year (e.g., 2020)**
   ```sql
   SELECT 
       netflix.title,
       netflix.release_year
   FROM 
       netflix
   WHERE
       release_year = '2020'
       AND type = 'Movie';
   ```

4. **Find the top 5 countries with the most content on Netflix**
   ```sql
   SELECT 
       UNNEST(STRING_TO_ARRAY(country, ',')) AS new_country,
       COUNT(show_id)
   FROM netflix
   GROUP BY 1
   ORDER BY 2 DESC
   LIMIT 5;
   ```

5. **Identify the longest movie**
   ```sql
   SELECT 
       title,
       MAX(CAST(SUBSTRING(duration, 1, POSITION(' ' IN duration)-1) AS INT)) AS maximun_lenght
   FROM netflix
   WHERE type = 'Movie' AS duration IS NOT NULL 
   GROUP BY 1
   ORDER BY 2 DESC
   LIMIT 1
   ```

6. **Find content added in the last 5 years**
   ```sql
   SELECT * FROM netflix
   WHERE to_date(date_added, 'Month DD, YYYY') >= CURRENT_DATE - interval '5 years';
   ```

7. **Find all the movies/TV shows by director 'Rajiv Chilaka'**
   ```sql
   SELECT 
       title,
       type,
       director,
       ROW_NUMBER() OVER()
   FROM netflix
   WHERE director like '%Rajiv Chilaka%';
   ```

8. **List all TV shows with more than 5 seasons**
   ```sql
   SELECT
       title,
       type,
       duration
   FROM netflix
   WHERE 
       type = 'TV Show'
       AND CAST(split_part(duration, ' ', 1) AS INT) > 5;
   ```

9. **Count the number of content items in each genre**
   ```sql
   SELECT 
       UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre,
       COUNT(*)
   FROM netflix
   GROUP BY 1
   ORDER BY 2 DESC;
   ```

10. **Find each year and the average numbers of content release in India on Netflix**
    ```sql
    SELECT 
        EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD, YYYY')),
        COUNT(*),
        COUNT(*)::numeric/(SELECT COUNT(*) FROM netflix WHERE country LIKE 'India')::numeric * 100 AS avg_content_per_year
    FROM netflix
    WHERE country LIKE 'India'
    GROUP BY 1
    ```

11. **List all movies that are documentaries**
    ```sql
    SELECT 
        *
    FROM netflix
    WHERE 
        listed_in ILIKE 'documentaries'
        AND type = 'Movie'
    ```

12. **Find all content without a director**
    ```sql
    SELECT
        *
    FROM netflix
    WHERE director IS NULL;
    ```

13. **Find how many movies actor 'Salman Khan' appeared in last 10 years**
    ```sql
    SELECT 
        *
    FROM netflix
    WHERE casts LIKE '%Salman Khan%'
    AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 11;
    ```

14. **Find the top 10 actors who have appeared in the highest number of movies produced in India**
    ```sql
    SELECT 
        UNNEST(STRING_TO_ARRAY(casts, ',')) as actors,
        COUNT(*) AS total_content
    FROM netflix 
    WHERE country ILIKE '%india%'
    GROUP BY 1
    ORDER BY 2 DESC
    LIMIT 10;
    ```

15. **Categorize the content based on the presence of the keywords 'kill' and 'violence' in the description field**
    ```sql
    WITH new_table
    AS
    (
        SELECT 
            *,
         CASE 
            WHEN description ILIKE '%kill%' OR 
                 description ILIKE '%violence%' THEN 'Bad' 
            ELSE 'Good'
         END category
        FROM netflix
    )
    SELECT
        category,
        COUNT(*)
    FROM new_table
    GROUP BY 1;
    ```

## How to Use

1. Import the Netflix dataset into your PostgreSQL database
2. Run the provided SQL queries to solve each business problem
3. Analyze the results to gain insights about the Netflix catalog

## Requirements

- PostgreSQL installed
- Netflix dataset loaded into the database
- Basic SQL knowledge to interpret and modify the queries

## Notes

Some queries may need adjustments depending on the exact structure of your dataset, especially regarding date formats and specific data types.
