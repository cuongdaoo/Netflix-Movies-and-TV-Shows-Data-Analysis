# **Netflix Movies and TV Shows Data Analysis**  
![image](https://github.com/user-attachments/assets/109aa11e-394d-4a9e-a7a4-23cb4b433c4b)


# **1. Overview**  
Netflix is one of the world's leading streaming platforms, offering a vast library of movies and TV shows across various genres and countries. With increasing competition in the streaming industry, understanding content trends, audience preferences, and production strategies is crucial for business growth and user engagement.  

This project aims to analyze Netflix's dataset to uncover insights about:  
- Content distribution (movies vs. TV shows)  
- Release trends over the years  
- Popular genres and ratings  
- Leading countries in content production  
- Directors and actors with the most contributions  

By leveraging data analysis techniques, we can identify patterns that help in decision-making for content creation, marketing strategies, and audience targeting.  

# **2. Objectives**  
The primary objectives of this analysis are:  
- **Explore the composition** of Netflix's catalog (movies vs. TV shows).  
- **Analyze growth trends** in content releases over time.  
- **Identify popular genres** and ratings to understand viewer preferences.  
- **Determine top-producing countries** to see where Netflix focuses its content creation.  
- **Examine key contributors** (directors, actors) who shape Netflix's library.  
- **Compare durations** of movies and TV show seasons to detect industry standards.  

This analysis will help stakeholders, including content creators, marketers, and business strategists, make data-driven decisions to enhance Netflix's offerings.  

# **3. Dataset Description**  
The dataset used in this project is sourced from **Kaggle**:  
📌 [Netflix Movies and TV Shows Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)  

## **Dataset Features:**  
- **show_id** – Unique identifier for each title  
- **type** – Whether the title is a "Movie" or "TV Show"  
- **title** – Name of the movie or TV show  
- **director** – Director(s) of the title  
- **cast** – Main actors featured in the title  
- **country** – Country where the title was produced  
- **date_added** – Date when the title was added to Netflix  
- **release_year** – Year the title was originally released  
- **rating** – Content rating (e.g., PG-13, TV-MA)  
- **duration** – Length of the movie (in minutes) or number of seasons (for TV shows)  
- **listed_in** – Genre(s) of the title  
- **description** – Brief summary of the plot  

# **Potential Analysis Questions:**  

## Data Type Conversion for date_added Column
**Question** - How to convert the data type of date_added column from string to DATE?

**Explanation**:
1. First, we add a new column with DATE data type
2. Then we attempt to convert/cast the existing string data to DATE format
3. Optional steps include:
   - Dropping the old column if no longer needed
   - Renaming the new column to match the original column name
4. The TRY_CAST function is used for safe conversion that returns NULL instead of error if conversion fails

**SQL Code**:
```sql
-- Add new column with DATE type
ALTER TABLE netflix_titles
ADD NewDate DATE;

-- Update new column with converted data
UPDATE netflix_titles
SET NewDate = TRY_CAST(date_added AS DATE);

-- Optional: Drop old column
ALTER TABLE netflix_titles
DROP COLUMN date_added;

-- Optional: Rename new column
EXEC sp_rename 'netflix_titles.NewDate', 'date_added', 'COLUMN';

-- Verify results
SELECT * FROM netflix_titles;
```

**Answer**:
[To be filled]

---

## Question 1
**Question** - Count the Number of Movies vs TV Shows

**Explanation**:
Two approaches are shown:
1. Using conditional SUM to count movies and TV shows in a single row
2. Using GROUP BY to get counts in separate rows
Both methods filter the dataset by type and provide counts for each category

**SQL Code**:
```sql
-- Method 1: Conditional sum
SELECT 
    SUM(CASE WHEN type = 'Movie' THEN 1 END) AS movie_count, 
    SUM(CASE WHEN type = 'TV Show' THEN 1 END) AS tvshow_count 
FROM netflix_titles;

-- Method 2: Group by
SELECT 
    type,
    COUNT(*) AS type_count
FROM netflix_titles
GROUP BY type;
```

**Answer**:
![image](https://github.com/user-attachments/assets/731f8d18-6609-418c-9313-204c320f8991)


---

## Question 2
**Question** - Find the Most Common Rating for Movies and TV Shows

**Explanation**:
1. First CTE calculates count for each type-rating combination
2. Second CTE ranks ratings within each type by count
3. Final query filters to only show top-ranked (most common) ratings for each type

**SQL Code**:
```sql
WITH cte AS (
    SELECT
        type,
        rating,
        COUNT(*) AS rating_count
    FROM netflix_titles
    GROUP BY type, rating
),
cte2 AS (
    SELECT 
        *, 
        RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS ranking 
    FROM cte
)
SELECT * FROM cte2
WHERE ranking = 1;
```

**Answer**:
![image](https://github.com/user-attachments/assets/a5746e96-c3f2-4210-8dd6-9dadbde43d82)


--- 
## Question 3  
**Question** - List All Movies Released in a Specific Year (e.g., 2020)  

**Explanation**:  
This query filters the dataset to show only movies released in the specified year (2020 in this example).  
- Uses a simple WHERE clause to filter by both `type` and `release_year`  
- The `release_year` field is stored as a string, so we compare it with a string value  

**SQL Code**:  
```sql
SELECT * 
FROM netflix_titles
WHERE release_year = '2020';
```

**Answer**:  
![image](https://github.com/user-attachments/assets/9b487dd5-9759-4979-b1e1-7bde4ac0dec2)


---

## Question 4  
**Question** - Find the Top 5 Countries with the Most Content on Netflix  

**Explanation**:  
1. Filters out records where country is NULL  
2. Groups by country and counts the number of titles  
3. Orders results by content count in descending order  
4. Uses TOP 5 to limit to the highest-ranking countries  

**SQL Code**:  
```sql
SELECT TOP 5 
    country, 
    COUNT(title) AS count_content 
FROM netflix_titles
WHERE country IS NOT NULL
GROUP BY country
ORDER BY count_content DESC;
```

**Answer**:  
![image](https://github.com/user-attachments/assets/b98a9b6e-09e1-4684-bd05-cc03850cde7b)
 

---

## Question 5  
**Question** - Identify the Longest Movie  

**Explanation**:  
1. Uses CROSS APPLY with STRING_SPLIT to break down the duration string (e.g., "90 min")  
2. Filters for only Movie type and excludes the 'min' value  
3. Converts the remaining value to integer  
4. Orders by duration in descending order and takes the top result  

**SQL Code**:  
```sql
WITH cte AS (
    SELECT * 
    FROM netflix_titles
    CROSS APPLY STRING_SPLIT(duration, ' ')
    WHERE type = 'Movie' AND value != 'min'
)
SELECT TOP 1 
    title, 
    CAST(value AS INT) AS duration_split 
FROM cte
ORDER BY duration_split DESC;
```

**Answer**:  
![image](https://github.com/user-attachments/assets/7e19e648-3c47-46f8-b3d5-2e844bcdc5fb)
 

---

## Question 6  
**Question** - Find Content Added in the Last 5 Years  

**Explanation**:  
1. Uses DATEDIFF function to calculate the difference between current date and date_added  
2. Filters for content where this difference equals 5 years  
3. Assumes the date_added column has been properly converted to DATE type  

**SQL Code**:  
```sql
SELECT * 
FROM netflix_titles
WHERE DATEDIFF(YEAR, date_added, GETDATE()) = 5;
```

**Answer**:  
![image](https://github.com/user-attachments/assets/a7d4e698-8c5e-4db2-984b-ccc578d04265)

---

## Question 7  
**Question** - Find All Movies/TV Shows by Director 'Rajiv Chilaka'  

**Explanation**:  
Simple filtering query that:  
1. Looks for exact match of director name  
2. Returns all columns for matching records  

**SQL Code**:  
```sql
SELECT * 
FROM netflix_titles
WHERE director = 'Rajiv Chilaka';
```

**Answer**:  
![image](https://github.com/user-attachments/assets/f3dc6aeb-1ee0-4e8d-bccf-11f6eb5de264)
 

---

## Question 8  
**Question** - List All TV Shows with More Than 5 Seasons  

**Explanation**:  
1. Splits the duration string for TV Shows (e.g., "3 Seasons")  
2. Excludes the words 'Season' and 'Seasons'  
3. Converts the numeric part to decimal  
4. Filters for shows with more than 5 seasons  

**SQL Code**:  
```sql
WITH cte AS (
    SELECT *
    FROM netflix_titles
    CROSS APPLY STRING_SPLIT(duration, ' ')
    WHERE type = 'TV Show' 
      AND value NOT IN ('Season', 'Seasons')
),
cte2 AS (
    SELECT 
        title, 
        TRY_CAST(value AS DECIMAL) AS duration_split
    FROM cte
    WHERE ISNUMERIC(value) = 1
)
SELECT *
FROM cte2
WHERE duration_split > 5
ORDER BY duration_split DESC;
```

**Answer**:  
![image](https://github.com/user-attachments/assets/767bd0d8-a0a1-4d87-985a-58c8fe9f272d)


---

## Question 9  
**Question** - Count the Number of Content Items in Each Genre  

**Explanation**:  
1. Splits the listed_in column by commas to separate individual genres  
2. Trims whitespace from genre names  
3. Groups by genre and counts occurrences  
4. Orders by count in descending order  

**SQL Code**:  
```sql
WITH cte AS (
    SELECT *, TRIM(value) AS genre
    FROM netflix_titles
    CROSS APPLY STRING_SPLIT(listed_in, ',')
)
SELECT 
    genre, 
    COUNT(show_id) AS value_count 
FROM cte
GROUP BY genre
ORDER BY value_count DESC;
```

**Answer**:  
![image](https://github.com/user-attachments/assets/2b7aa5c8-a916-47c7-bda8-145b6c035e9a)


---

## Question 10  
**Question** - Find each year and the average numbers of content release in India on Netflix  

**Explanation**:  
1. Filters for content from India  
2. Groups by release year  
3. Calculates both count and percentage of total Indian content  
4. Uses ROUND to format the percentage to 2 decimal places  

**SQL Code**:  
```sql
SELECT
    country,
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(
        CAST(COUNT(show_id) AS NUMERIC) /
        CAST((SELECT COUNT(show_id) FROM netflix_titles WHERE country = 'India') AS NUMERIC) * 100,
        2
    ) AS avg_release
FROM netflix_titles
WHERE country = 'India'
GROUP BY country, release_year
ORDER BY avg_release DESC;
```

**Answer**:  
![image](https://github.com/user-attachments/assets/cbfc76aa-1b94-4bb7-a90a-569112b935c5)
  


---

## Question 11  
**Question** - List All Movies that are Documentaries  

**Explanation**:  
1. Splits the `listed_in` column to separate genres  
2. Trims whitespace from genre values  
3. Filters for entries where genre exactly matches 'Documentaries'  
4. Returns all columns for matching records  

**SQL Code**:  
```sql
WITH cte AS (
    SELECT *, TRIM(value) AS genre
    FROM netflix_titles
    CROSS APPLY STRING_SPLIT(listed_in, ',')
)
SELECT * FROM cte
WHERE genre = 'Documentaries';
```

**Answer**:  
![image](https://github.com/user-attachments/assets/7ddd88c2-e3b1-421a-b413-552eaffcfa73)
  

---

## Question 12  
**Question** - Find All Content Without a Director  

**Explanation**:  
Simple query that:  
1. Checks for NULL values in the director column  
2. Returns all content where director information is missing  

**SQL Code**:  
```sql
SELECT * 
FROM netflix_titles
WHERE director IS NULL;
```

**Answer**:  
![image](https://github.com/user-attachments/assets/deed4fc2-0dba-41a2-b306-b5f448482e0a)
  

---

## Question 13  
**Question** - Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years  

**Explanation**:  
1. Splits the cast column to separate individual actors  
2. Trims whitespace from actor names  
3. Filters for:  
   - Actor name exactly matching 'Salman Khan'  
   - Content released within last 10 years (current year minus release year ≤ 10)  

**SQL Code**:  
```sql
WITH cte AS (
    SELECT *, TRIM(value) AS actor
    FROM netflix_titles
    CROSS APPLY STRING_SPLIT(cast, ',')
)
SELECT * FROM cte
WHERE actor = 'Salman Khan' 
  AND YEAR(GETDATE()) - release_year <= 10;
```

**Answer**:  
![image](https://github.com/user-attachments/assets/7326dbe1-1dcd-4af4-b52c-619710dbcf46)


---

## Question 14  
**Question** - Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India  

**Explanation**:  
1. Splits cast column to separate actors  
2. Trims whitespace from actor names  
3. Filters for content produced in India  
4. Groups by actor and counts appearances  
5. Orders by appearance count in descending order  
6. Limits to top 10 results  

**SQL Code**:  
```sql
WITH cte AS (
    SELECT *, TRIM(value) AS actor
    FROM netflix_titles
    CROSS APPLY STRING_SPLIT(cast, ',')
)
SELECT TOP 10 
    actor, 
    COUNT(*) AS actor_count 
FROM cte
WHERE country = 'India'
GROUP BY actor
ORDER BY actor_count DESC;
```

**Answer**:  
![image](https://github.com/user-attachments/assets/e7ddf838-2aa1-4595-b608-1c3b5c5adf26)
 

---

## Question 15  
**Question** - Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords  

**Explanation**:  
1. Creates a CTE that adds a category column  
2. Uses CASE statement to classify content as:  
   - 'Bad' if description contains 'kill' or 'violence'  
   - 'Good' otherwise  
3. Main query counts items in each category  
4. Results ordered by count in descending order  

**SQL Code**:  
```sql
WITH cte AS (
    SELECT 
        *, 
        CASE 
            WHEN description LIKE '%kill%' OR description LIKE '%violence%' THEN 'Bad' 
            ELSE 'Good' 
        END AS category 
    FROM netflix_titles
)
SELECT 
    category, 
    COUNT(*) AS cate_count 
FROM cte
GROUP BY category
ORDER BY cate_count DESC;
```

**Answer**:  
![image](https://github.com/user-attachments/assets/8465269d-a84f-4b03-be8a-3a1e85fcf6af)


---
