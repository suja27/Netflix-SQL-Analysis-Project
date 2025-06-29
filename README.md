# 🎬 Netflix SQL Project

This project is a comprehensive data exploration of Netflix's movie and TV show catalog using **SQL Server**. It aims to derive actionable insights from the dataset such as content type trends, popular genres, and country-level production patterns.

## Netflix SQL Project Overview
This project provides a comprehensive SQL-based analysis of Netflix’s catalog of movies and TV shows. The primary objective is to extract meaningful business insights from the data, identify trends, and support decision-making with evidence-based findings. The analysis is performed using SQL queries on a cleaned and structured dataset.

 ## 🎯 Project Objectives
- Analyze the distribution of content types (Movies vs. TV Shows).

- Identify the most common ratings applied to different types of content.

- Explore release trends over the years and across countries.

- Filter and categorize content using specific criteria (e.g., genre, duration, keywords).

- Understand which countries produce the most content on Netflix.



## 🧱 Table Schema

```
Sql

IF OBJECT_ID('dbo.netflix', 'U') IS NOT NULL
    DROP TABLE dbo.netflix;

CREATE TABLE dbo.netflix
(
    show_id      VARCHAR(10),
    type         VARCHAR(20),
    title        VARCHAR(250),
    director     VARCHAR(500),
    casts        VARCHAR(1000),
    country      VARCHAR(250),
    date_added   VARCHAR(50),
    release_year INT,
    rating       VARCHAR(10),
    duration     VARCHAR(20),
    listed_in    VARCHAR(250),
    description  VARCHAR(500)
);

```

##  Business Problems and Solutions

 **1. Count of Movies vs TV Shows**
```
SELECT type, COUNT(*) AS total_count
FROM netflix
GROUP BY type;
```
**2. Top 10 Countries by Number of Titles**
```
SELECT country, COUNT(*) AS title_count
FROM netflix
WHERE country IS NOT NULL
GROUP BY country
ORDER BY title_count DESC
OFFSET 0 ROWS FETCH NEXT 10 ROWS ONLY;

-- Most Common Ratings
SELECT rating, COUNT(*) AS count
FROM netflix
GROUP BY rating
ORDER BY count DESC;

SELECT rating, COUNT(*) AS count
FROM netflix
GROUP BY rating
ORDER BY count DESC;


-- Most Common Ratings
SELECT rating, COUNT(*) AS count
FROM netflix
GROUP BY rating
ORDER BY count DESC;

SELECT 
    DATENAME(MONTH, TRY_CAST(date_added AS DATE)) AS month_name,
    COUNT(*) AS titles_added
FROM netflix
WHERE date_added IS NOT NULL
GROUP BY DATENAME(MONTH, TRY_CAST(date_added AS DATE))
ORDER BY titles_added DESC;

--Longest Duration Shows
 SELECT TOP 10 title, duration, type
FROM netflix
WHERE duration LIKE '%min%' -- For Movies
   OR duration LIKE '%Season%'
ORDER BY 
   CASE 
      WHEN duration LIKE '%Season%' THEN TRY_CAST(LEFT(duration, CHARINDEX(' ', duration)-1) AS INT)
      ELSE TRY_CAST(LEFT(duration, CHARINDEX(' ', duration)-1) AS INT)
   END DESC;

--   TV Shows vs Movies Over the Years

SELECT release_year, type, COUNT(*) AS total
FROM netflix
GROUP BY release_year, type
ORDER BY release_year, type;

--  List All Movies that are Documentaries
SELECT * 
FROM netflix
WHERE listed_in LIKE '%Documentaries%';

--Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years
SELECT * 
FROM netflix
WHERE casts LIKE '%Salman Khan%'
  AND release_year > YEAR(GETDATE()) - 10;

 -- Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India

SELECT TOP 10 
    TRIM(value) AS actor,
    COUNT(*) AS appearances
FROM netflix
CROSS APPLY STRING_SPLIT(casts, ',')
WHERE country = 'India'
GROUP BY TRIM(value)
ORDER BY appearances DESC;


--Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords
SELECT 
    CASE 
        WHEN description LIKE '%kill%' OR description LIKE '%violence%' THEN 'Bad'
        ELSE 'Good'
    END AS category,
    COUNT(*) AS content_count
FROM netflix
GROUP BY 
    CASE 
        WHEN description LIKE '%kill%' OR description LIKE '%violence%' THEN 'Bad'
        ELSE 'Good'
    END;





