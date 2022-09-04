### DataCamp_Project-Analyzing-Video-Games-Sales
In this project, we'll explore the top 400 best-selling video games created between 1977 and 2020. We'll compare a dataset on game sales with critic and user reviews to determine whether or not video games have improved as the gaming market has grown.

Our database contains two tables:


#### game_sales
```markdown
|   column   |  type   |             meaning              |
|------------|---------|----------------------------------|
| game       | varchar | Name of the video game           |
| platform   | varchar | Gaming platform                  |
| publisher  | varchar | Game publisher                   |
| developer  | varchar | Game developer                   |
| games_sold | float   | Number of copies sold (millions) |
| year       | int     | Release year                     |
 ```
 
#### reviews
```markdown
|    column    |  type   |               meaning                |
|--------------|---------|--------------------------------------|
| game         | varchar | Name of the video game               |
| critic_score | float   | Critic score according to Metacritic |
| user_score   | float   | User score according to Metacritic   |
 ``` 
 
 --------------------------
 
 #### 1. Let's begin by looking at some of the top selling video games of all time!
 
 ```sql
SELECT *
FROM game_sales
ORDER BY games_sold desc
LIMIT 10;
```

#### Output:

```markdown

|                   game                    | platform |    publisher     |     developer     | games_sold | year |
|-------------------------------------------|----------|------------------|-------------------|------------|------|
| Wii Sports for Wii                        | Wii      | Nintendo         | Nintendo EAD      |      82.90 | 2006 |
| Super Mario Bros. for NES                 | NES      | Nintendo         | Nintendo EAD      |      40.24 | 1985 |
| Counter-Strike: Global Offensive for PC   | PC       | Valve            | Valve Corporation |      40.00 | 2012 |
| Mario Kart Wii for Wii                    | Wii      | Nintendo         | Nintendo EAD      |      37.32 | 2008 |
| PLAYERUNKNOWN'S BATTLEGROUNDS for PC      | PC       | PUBG Corporation | PUBG Corporation  |      36.60 | 2017 |
| Minecraft for PC                          | PC       | Mojang           | Mojang AB         |      33.15 | 2010 |
| Wii Sports Resort for Wii                 | Wii      | Nintendo         | Nintendo EAD      |      33.13 | 2009 |
| Pokemon Red / Green / Blue Version for GB | GB       | Nintendo         | Game Freak        |      31.38 | 1998 |
| New Super Mario Bros. for DS              | DS       | Nintendo         | Nintendo EAD      |      30.80 | 2006 |
| New Super Mario Bros. Wii for Wii         | Wii      | Nintendo         | Nintendo EAD      |      30.30 | 2009 |

```
----------------------
#### 2.  Missing review scores

Wow, the best-selling video games were released between 1985 to 2017! That's quite a range; we'll have to use data from the `reviews` table to gain more insight on the best years for video games.

First, it's important to explore the limitations of our database. One big shortcoming is that there is not any `reviews` data for some of the games on the `game_sales` table.

```sql
SELECT COUNT(*) --Counts the number of games
FROM game_sales AS g
LEFT JOIN reviews AS r
ON g.game = r.game
WHERE critic_score IS NULL --only filter data where there is no scores or missing scores
    AND user_score IS NULL
```

#### Output:

```markdown
| count |
|-------|
|    31 |
```
----------------------
#### 3. Years that video game critics loved

It looks like a little less than ten percent of the games on the `game_sales` table don't have any reviews data. That's a small enough percentage that we can continue our exploration, but the missing reviews data is a good thing to keep in mind as we move on to evaluating results from more sophisticated queries.

There are lots of ways to measure the best years for video games! Let's start with what the critics think.

```sql
SELECT 
    year
    ,ROUND (AVG (critic_score), 2) AS avg_critic_score
FROM game_sales
INNER JOIN reviews
USING (game)
GROUP BY year
ORDER BY avg_critic_score DESC
LIMIT 10;
```
#### Output:

```markdown

| year | avg_critic_score |
|------|------------------|
| 1990 |             9.80 |
| 1992 |             9.67 |
| 1998 |             9.32 |
| 2020 |             9.20 |
| 1993 |             9.10 |
| 1995 |             9.07 |
| 2004 |             9.03 |
| 1982 |             9.00 |
| 2002 |             8.99 |
| 1999 |             8.93 |

```
----------------------
#### 4. Was 1982 really that great?

The range of great years according to critic reviews goes from 1982 until 2020: we are no closer to finding the golden age of video games!

Hang on, though. Some of those `avg_critic_score` values look like suspiciously round numbers for averages. The value for 1982 looks especially fishy. Maybe there weren't a lot of video games in our dataset that were released in certain years.

Let's update our query and find out whether 1982 really was such a great year for video games.

```sql	
SELECT 
    year
    ,COUNT (*) AS num_games
    ,ROUND (AVG (critic_score), 2) AS avg_critic_score
FROM game_sales
INNER JOIN reviews
USING (game)
GROUP BY year
HAVING COUNT (*) > 4 --Only returns years that have more than four reviewed games
ORDER BY avg_critic_score DESC
LIMIT 10;
```

#### Output:

```markdown
| year | num_games | avg_critic_score |
|------|-----------|------------------|
| 1998 |        10 |             9.32 |
| 2004 |        11 |             9.03 |
| 2002 |         9 |             8.99 |
| 1999 |        11 |             8.93 |
| 2001 |        13 |             8.82 |
| 2011 |        26 |             8.76 |
| 2016 |        13 |             8.67 |
| 2013 |        18 |             8.66 |
| 2008 |        20 |             8.63 |
| 2017 |        13 |             8.62 |
```
----------------------
#### 5. Years that dropped off the critics' favorites list

That looks better! The `num_games` column convinces us that our new list of the critics' top games reflects years that had quite a few well-reviewed games rather than just one or two hits. But which years dropped off the list due to having four or fewer reviewed games? Let's identify them so that someday we can track down more game reviews for those years and determine whether they might rightfully be considered as excellent years for video game releases!

```sql
WITH top_critic_years AS (
    SELECT 
        year
        ,ROUND (AVG (critic_score), 2) AS avg_critic_score
    FROM game_sales
    INNER JOIN reviews
    USING (game)
    GROUP BY year
    ),


  top_critic_years_more_than_four_games AS (
    SELECT 
        year
        ,COUNT (*) AS num_games
        ,ROUND (AVG (critic_score), 2) AS avg_critic_score
    FROM game_sales
    INNER JOIN reviews
    USING (game)
    GROUP BY year
    HAVING COUNT (*) > 4
    )
```

```sql
SELECT 
    year
    ,avg_critic_score
FROM top_critic_years

EXCEPT

SELECT 
    year
    ,avg_critic_score
FROM top_critic_years_more_than_four_games
ORDER BY avg_critic_score DESC
```

#### Output:

```markdown
| year | avg_critic_score |
|------|------------------|
| 1990 |             9.80 |
| 1992 |             9.67 |
| 2020 |             9.20 |
| 1993 |             9.10 |
| 1995 |             9.07 |
| 1982 |             9.00 |
```

