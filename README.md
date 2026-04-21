# 🎬 Netflix Data Analysis using SQL (End-to-End Project)

## 📌 Overview

This project performs a **complete SQL-based analysis of Netflix Movies & TV Shows dataset**, covering **basic to advanced SQL concepts** including filtering, aggregation, string manipulation, CTEs, and window functions.

It is designed to showcase **real-world SQL problem-solving skills** required for Data Analyst roles.

---

## 🎯 Objectives

* Analyze content distribution
* Identify top actors, directors, countries
* Understand content trends over time
* Perform genre & keyword analysis
* Apply advanced SQL techniques

---

## 🧱 Schema

```sql
DROP TABLE IF EXISTS netflix;

CREATE TABLE netflix (
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);
```

---

# 🧠 SQL Questions & Solutions

---

## 🟢 BASIC (1–10)

### 1. What is the total number of records in the dataset?

```sql
SELECT COUNT(*) FROM netflix;
```

### 2. How many Movies and TV Shows are there?

```sql
SELECT type, COUNT(*) 
FROM netflix 
GROUP BY type;
```

### 3. What are the unique ratings available?

```sql
SELECT DISTINCT rating 
FROM netflix;
```

### 4. List all content released in 2020

```sql
SELECT * 
FROM netflix 
WHERE release_year = 2020;
```

### 5. Show the latest 5 contents added to Netflix

```sql
SELECT * 
FROM netflix
ORDER BY TO_DATE(date_added,'Month DD, YYYY') DESC
LIMIT 5;
```

### 6. Count number of content items per rating

```sql
SELECT rating, COUNT(*) 
FROM netflix 
GROUP BY rating;
```

### 7. Retrieve all Movies

```sql
SELECT * 
FROM netflix 
WHERE type = 'Movie';
```

### 8. Retrieve all TV Shows

```sql
SELECT * 
FROM netflix 
WHERE type = 'TV Show';
```

### 9. How many distinct countries are present?

```sql
SELECT COUNT(DISTINCT country) 
FROM netflix;
```

### 10. Find all records where director is missing

```sql
SELECT * 
FROM netflix 
WHERE director IS NULL;
```

---

## 🟡 INTERMEDIATE (11–25)

### 11. What is the most common rating for each content type?

```sql
WITH cte AS (
SELECT type, rating, COUNT(*) cnt
FROM netflix
GROUP BY type, rating
)
SELECT *
FROM (
SELECT *, RANK() OVER(PARTITION BY type ORDER BY cnt DESC) rnk
FROM cte
) t
WHERE rnk = 1;
```

---

### 12. Which are the top 5 countries with most content?

```sql
SELECT *
FROM (
SELECT UNNEST(STRING_TO_ARRAY(country,',')) country, COUNT(*) cnt
FROM netflix
GROUP BY 1
) t
ORDER BY cnt DESC
LIMIT 5;
```

---

### 13. How many contents were added each year?

```sql
SELECT EXTRACT(YEAR FROM TO_DATE(date_added,'Month DD, YYYY')) year,
COUNT(*)
FROM netflix
GROUP BY year
ORDER BY year;
```

---

### 14. Which is the longest movie?

```sql
SELECT *
FROM netflix
WHERE type='Movie'
ORDER BY SPLIT_PART(duration,' ',1)::INT DESC
LIMIT 1;
```

---

### 15. Which is the shortest movie?

```sql
SELECT *
FROM netflix
WHERE type='Movie'
ORDER BY SPLIT_PART(duration,' ',1)::INT
LIMIT 1;
```

---

### 16. How many content items exist per genre?

```sql
SELECT UNNEST(STRING_TO_ARRAY(listed_in,',')) genre,
COUNT(*)
FROM netflix
GROUP BY genre;
```

---

### 17. List all documentary content

```sql
SELECT *
FROM netflix
WHERE listed_in ILIKE '%Documentaries%';
```

---

### 18. Find content with missing cast information

```sql
SELECT *
FROM netflix
WHERE casts IS NULL;
```

---

### 19. Which directors have created the most content?

```sql
SELECT director, COUNT(*)
FROM netflix
GROUP BY director
ORDER BY COUNT(*) DESC;
```

---

### 20. Who are the top 10 most frequent actors?

```sql
SELECT UNNEST(STRING_TO_ARRAY(casts,',')) actor,
COUNT(*) cnt
FROM netflix
GROUP BY actor
ORDER BY cnt DESC
LIMIT 10;
```

---

### 21. How much content is produced in India each year?

```sql
SELECT release_year, COUNT(*)
FROM netflix
WHERE country ILIKE '%India%'
GROUP BY release_year
ORDER BY release_year;
```

---

### 22. What content was added in the last 5 years?

```sql
SELECT *
FROM netflix
WHERE TO_DATE(date_added,'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
```

---

### 23. Split and normalize country values

```sql
SELECT show_id, UNNEST(STRING_TO_ARRAY(country,',')) country
FROM netflix;
```

---

### 24. Which TV shows have more than 3 seasons?

```sql
SELECT *
FROM netflix
WHERE type='TV Show'
AND SPLIT_PART(duration,' ',1)::INT > 3;
```

---

### 25. What is the average release year per content type?

```sql
SELECT type, AVG(release_year)
FROM netflix
GROUP BY type;
```

---

## 🔵 ADVANCED (26–45)

---

### 26. Rank actors based on number of appearances

```sql
SELECT actor, cnt,
RANK() OVER(ORDER BY cnt DESC)
FROM (
SELECT UNNEST(STRING_TO_ARRAY(casts,',')) actor,
COUNT(*) cnt
FROM netflix
GROUP BY actor
) t;
```

---

### 27. Which cast combinations appear most frequently?

```sql
SELECT casts, COUNT(*)
FROM netflix
GROUP BY casts
ORDER BY COUNT(*) DESC
LIMIT 10;
```

---

### 28. How has content growth changed over the years?

```sql
SELECT release_year, COUNT(*)
FROM netflix
GROUP BY release_year
ORDER BY release_year;
```

---

### 29. What is the percentage contribution of each country?

```sql
SELECT country,
COUNT(*)*100.0/(SELECT COUNT(*) FROM netflix) pct
FROM netflix
GROUP BY country;
```

---

### 30. What is the year-over-year growth of content?

```sql
SELECT year,
total,
total - LAG(total) OVER(ORDER BY year) growth
FROM (
SELECT release_year year, COUNT(*) total
FROM netflix
GROUP BY release_year
) t;
```

---

### 31. Are there duplicate titles?

```sql
SELECT title, COUNT(*)
FROM netflix
GROUP BY title
HAVING COUNT(*) > 1;
```

---

### 32. Categorize content as Old vs Modern

```sql
SELECT 
CASE WHEN release_year < 2000 THEN 'Old' ELSE 'Modern' END category,
COUNT(*)
FROM netflix
GROUP BY category;
```

---

### 33. How often does the keyword "love" appear?

```sql
SELECT COUNT(*)
FROM netflix
WHERE description ILIKE '%love%';
```

---

### 34. Classify content as Violent or Family-friendly

```sql
SELECT 
CASE 
WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Violent'
ELSE 'Family'
END,
COUNT(*)
FROM netflix
GROUP BY 1;
```

---

### 35. What are top genres per country?

```sql
SELECT country, genre, COUNT(*)
FROM (
SELECT UNNEST(STRING_TO_ARRAY(country,',')) country,
UNNEST(STRING_TO_ARRAY(listed_in,',')) genre
FROM netflix
) t
GROUP BY country, genre;
```

---

### 36. Which directors work across multiple countries?

```sql
SELECT director, COUNT(DISTINCT country)
FROM netflix
GROUP BY director
ORDER BY 2 DESC;
```

---

### 37. What is the average movie duration?

```sql
SELECT AVG(SPLIT_PART(duration,' ',1)::INT)
FROM netflix
WHERE type='Movie';
```

---

### 38. How are ratings distributed across types?

```sql
SELECT type, rating, COUNT(*)
FROM netflix
GROUP BY type, rating;
```

---

### 39. Which shows are binge-worthy (≥5 seasons)?

```sql
SELECT *
FROM netflix
WHERE type='TV Show'
AND SPLIT_PART(duration,' ',1)::INT >= 5;
```

---

### 40. What are the most common genre combinations?

```sql
SELECT listed_in, COUNT(*)
FROM netflix
GROUP BY listed_in
ORDER BY COUNT(*) DESC;
```

---

### 41. Analyze missing data patterns

```sql
SELECT 
COUNT(*) FILTER (WHERE director IS NULL) director_null,
COUNT(*) FILTER (WHERE casts IS NULL) cast_null
FROM netflix;
```

---

### 42. Which decade produced the most content?

```sql
SELECT (release_year/10)*10 decade, COUNT(*)
FROM netflix
GROUP BY decade
ORDER BY COUNT(*) DESC;
```

---

### 43. Create rolling count of content over years

```sql
SELECT release_year,
COUNT(*) OVER(ORDER BY release_year)
FROM netflix;
```

---

### 44. Which genre is most popular overall?

```sql
SELECT genre, COUNT(*)
FROM (
SELECT UNNEST(STRING_TO_ARRAY(listed_in,',')) genre
FROM netflix
) t
GROUP BY genre
ORDER BY COUNT(*) DESC;
```

---

### 45. Segment content into Premium vs General

```sql
SELECT 
CASE 
WHEN rating IN ('TV-MA','R') THEN 'Premium'
ELSE 'General'
END segment,
COUNT(*)
FROM netflix
GROUP BY segment;
```

---

## 📊 Key Insights

* Movies dominate Netflix content
* USA & India lead production
* Content growth accelerated after 2015
* TV-MA is the most common rating
* Multi-country collaborations are increasing

---

## 🚀 Project Highlights

✔ Covers Basic → Advanced SQL
✔ Real-world dataset handling
✔ Business-focused queries
✔ Interview-ready portfolio project

---

## 👨‍💻 Author

**Shubham Khanvilkar**
SQL | Data Analytics | Six Sigma Certified
