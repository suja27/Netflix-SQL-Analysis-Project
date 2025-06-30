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

```sql

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
## Data Cleaning 
--Remove duplicates 
```sql
select show_id,COUNT(*) 
from netflix
group by show_id 
having COUNT(*)>1

select * from netflix
where concat(upper(title),type)  in (
select concat(upper(title),type) 
from netflix_raw
group by upper(title) ,type
having COUNT(*)>1
)
order by title

with cte as (
select * 
,ROW_NUMBER() over(partition by title , type order by show_id) as rn
from netflix
)
select show_id,type,title,cast(date_added as date) as date_added,release_year
,rating,case when duration is null then rating else duration end as duration,description
into netflix
from cte

--populate missing values in country,duration columns
insert into netflix_country
select  show_id,m.country 
from netflix nr
inner join (
select director,country
from  netflix_country nc
inner join netflix_directors nd on nc.show_id=nd.show_id
group by director,country
) m on nr.director=m.director
where nr.country is null

select director,country
from  netflix_country nc
inner join netflix_directors nd on nc.show_id=nd.show_id
group by director,country

```


## Netflix data analysis
```sql
/*1  for each director count the no of movies and tv shows created by them in separate columns 
for directors who have created tv shows and movies both */
select nd.director 
,COUNT(distinct case when n.type='Movie' then n.show_id end) as no_of_movies
,COUNT(distinct case when n.type='TV Show' then n.show_id end) as no_of_tvshow
from netflix n
inner join netflix_directors nd on n.show_id=nd.show_id
group by nd.director
having COUNT(distinct n.type)>1


--2 which country has highest number of comedy movies 
select  top 1 nc.country , COUNT(distinct ng.show_id ) as no_of_movies
from netflix_genre ng
inner join netflix_country nc on ng.show_id=nc.show_id
inner join netflix n on ng.show_id=nc.show_id
where ng.genre='Comedies' and n.type='Movie'
group by  nc.country
order by no_of_movies desc


--3 for each year (as per date added to netflix), which director has maximum number of movies released
with cte as (
select nd.director,YEAR(date_added) as date_year,count(n.show_id) as no_of_movies
from netflix n
inner join netflix_directors nd on n.show_id=nd.show_id
where type='Movie'
group by nd.director,YEAR(date_added)
)
, cte2 as (
select *
, ROW_NUMBER() over(partition by date_year order by no_of_movies desc, director) as rn
from cte
--order by date_year, no_of_movies desc
)
select * from cte2 where rn=1



--4 what is average duration of movies in each genre
select ng.genre , avg(cast(REPLACE(duration,' min','') AS int)) as avg_duration
from netflix n
inner join netflix_genre ng on n.show_id=ng.show_id
where type='Movie'
group by ng.genre

--5  find the list of directors who have created horror and comedy movies both.
-- display director names along with number of comedy and horror movies directed by them 
select nd.director
, count(distinct case when ng.genre='Comedies' then n.show_id end) as no_of_comedy 
, count(distinct case when ng.genre='Horror Movies' then n.show_id end) as no_of_horror
from netflix n
inner join netflix_genre ng on n.show_id=ng.show_id
inner join netflix_directors nd on n.show_id=nd.show_id
where type='Movie' and ng.genre in ('Comedies','Horror Movies')
group by nd.director
having COUNT(distinct ng.genre)=2;

select * from netflix_genre where show_id in 
(select show_id from netflix_directors where director='Steve Brill')
order by genre
```
##  Business Problems and Solutions

 **1. Count of Movies vs TV Shows**
```sql
SELECT type, COUNT(*) AS total_count
FROM netflix
GROUP BY type;
```
**2. Top 10 Countries by Number of Titles**
```sql
SELECT country, COUNT(*) AS title_count
FROM netflix
WHERE country IS NOT NULL
GROUP BY country
ORDER BY title_count DESC
```

**3.Most Common Ratings**
```sql
SELECT rating, COUNT(*) AS count
FROM netflix
GROUP BY rating
ORDER BY count DESC;
```
 **4.Titles Added by Month**
```sql
SELECT 
    DATENAME(MONTH, TRY_CAST(date_added AS DATE)) AS month_name,
    COUNT(*) AS titles_added
FROM netflix
WHERE date_added IS NOT NULL
GROUP BY DATENAME(MONTH, TRY_CAST(date_added AS DATE))
ORDER BY titles_added DESC;
```

**5.Longest Duration Shows**
```Sql
 SELECT TOP 10 title, duration, type
FROM netflix
WHERE duration LIKE '%min%' -- For Movies
   OR duration LIKE '%Season%'
ORDER BY 
   CASE 
      WHEN duration LIKE '%Season%' THEN TRY_CAST(LEFT(duration, CHARINDEX(' ', duration)-1) AS INT)
      ELSE TRY_CAST(LEFT(duration, CHARINDEX(' ', duration)-1) AS INT)
   END DESC;
```

**6.  TV Shows vs Movies Over the Years**
```Sql
SELECT release_year, type, COUNT(*) AS total
FROM netflix
GROUP BY release_year, type
ORDER BY release_year, type;
```

**7.  List All Movies that are Documentaries**
```sql
SELECT * 
FROM netflix
WHERE listed_in LIKE '%Documentaries%';
```
**8.Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years**

```sql
SELECT * 
FROM netflix
WHERE casts LIKE '%Salman Khan%'
  AND release_year > YEAR(GETDATE()) - 10;
'''

 **9.Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India**
```sql
SELECT TOP 10 
    TRIM(value) AS actor,
    COUNT(*) AS appearances
FROM netflix
CROSS APPLY STRING_SPLIT(casts, ',')
WHERE country = 'India'
GROUP BY TRIM(value)
ORDER BY appearances DESC;
```
**10.Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords**

```sql
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
```
**11.Top 10 most common genres for MOVIES**


```sql
SELECT TOP 10 genres, 
COUNT(*) AS title_count
FROM shows_movies.titles 
WHERE type = 'moview'
GROUP BY genres
ORDER BY title_count DESC
```

**12.Top 10 most common genres for SHOWS**
```sql
SELECT genres, 
COUNT(*) AS title_count
FROM shows_movies.titles 
WHERE type = 'Show'
GROUP BY genres
ORDER BY title_count DESC
```





