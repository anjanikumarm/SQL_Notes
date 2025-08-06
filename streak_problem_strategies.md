# SQL Streak Problems: General Strategies and Techniques

## Core Strategy Overview

Streak problems typically involve finding consecutive sequences in data. The key insight is to **identify groups of consecutive records** and then **analyze those groups**.

## Fundamental Approach

### 1. **Group Consecutive Records**
The core technique uses the mathematical property that for consecutive sequences, the difference between row numbers remains constant.

```sql
-- Basic pattern for grouping consecutive records
SELECT 
    *,
    ROW_NUMBER() OVER (ORDER BY date_column) - 
    ROW_NUMBER() OVER (PARTITION BY condition_column ORDER BY date_column) AS group_id
FROM table_name
```


 
## Detailed Techniques 

### Technique 1: Classic Streak Detection 

```sql

-- Example: Longest winning streak for a team
WITH streak_groups AS (
    SELECT 
        team_id,
        game_date,
        result,
        game_id,
        ROW_NUMBER() OVER (PARTITION BY team_id ORDER BY game_date) -
        ROW_NUMBER() OVER (PARTITION BY team_id, result ORDER BY game_date) AS group_id
    FROM games
    WHERE result = 'W'  -- Only wins
),
streak_lengths AS (
    SELECT 
        team_id,
        group_id,
        COUNT(*) as streak_length,
        MIN(game_date) as start_date,
        MAX(game_date) as end_date
    FROM streak_groups
    GROUP BY team_id, group_id
)
SELECT 
    team_id,
    MAX(streak_length) as longest_streak
FROM streak_lengths
GROUP BY team_id
ORDER BY longest_streak DESC;
```
 
### Technique 2: Complex Conditions 

```sql

-- Example: Longest binge-watching streak with minimum 2 hours per day
WITH daily_watching AS (
    SELECT 
        user_id,
        DATE(watch_time) as watch_date,
        SUM(duration) as total_duration
    FROM streaming_data
    GROUP BY user_id, DATE(watch_time)
    HAVING SUM(duration) >= 120  -- At least 2 hours
),
streak_groups AS (
    SELECT 
        user_id,
        watch_date,
        total_duration,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY watch_date) -
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY watch_date) AS group_id
    FROM daily_watching
),
streak_analysis AS (
    SELECT 
        user_id,
        group_id,
        COUNT(*) as streak_days,
        SUM(total_duration) as total_hours,
        MIN(watch_date) as streak_start,
        MAX(watch_date) as streak_end
    FROM streak_groups
    GROUP BY user_id, group_id
)
SELECT 
    user_id,
    streak_days as longest_streak,
    total_hours,
    streak_start,
    streak_end
FROM streak_analysis
ORDER BY streak_days DESC;

``` 
 
### Technique 3: Gap Detection (Non-Consecutive) 

```sql

-- Example: Streaks with maximum 1-day gaps allowed
WITH ordered_data AS (
    SELECT 
        user_id,
        activity_date,
        LAG(activity_date) OVER (PARTITION BY user_id ORDER BY activity_date) as prev_date
    FROM user_activities
),
gap_detection AS (
    SELECT 
        user_id,
        activity_date,
        CASE 
            WHEN prev_date IS NULL OR 
                 activity_date - prev_date > 2  -- More than 1 day gap
            THEN 1 
            ELSE 0 
        END as new_streak_start
    FROM ordered_data
),
streak_numbering AS (
    SELECT 
        user_id,
        activity_date,
        SUM(new_streak_start) OVER (
            PARTITION BY user_id 
            ORDER BY activity_date 
            ROWS UNBOUNDED PRECEDING
        ) as streak_id
    FROM gap_detection
)
SELECT 
    user_id,
    streak_id,
    COUNT(*) as streak_length,
    MIN(activity_date) as start_date,
    MAX(activity_date) as end_date
FROM streak_numbering
GROUP BY user_id, streak_id
ORDER BY streak_length DESC;

``` 
 
### Advanced Patterns 

#### Pattern 1: Multiple Conditions 

```sql
-- Example: Streak of games where team scored > 100 points AND won
WITH qualified_games AS (
    SELECT *
    FROM games
    WHERE points_scored > 100 AND result = 'W'
),
streak_groups AS (
    SELECT 
        *,
        ROW_NUMBER() OVER (PARTITION BY team_id ORDER BY game_date) -
        ROW_NUMBER() OVER (PARTITION BY team_id ORDER BY game_date) AS group_id
    FROM qualified_games
)
SELECT 
    team_id,
    COUNT(*) as qualified_streak_length,
    MIN(game_date) as streak_start,
    MAX(game_date) as streak_end
FROM streak_groups
GROUP BY team_id, group_id
ORDER BY qualified_streak_length DESC;
``` 
 
#### Pattern 2: Rolling Window Streaks 

```sql
 
-- Example: 7-day rolling streak detection
WITH daily_activity AS (
    SELECT 
        user_id,
        activity_date,
        COUNT(*) as daily_count
    FROM user_logs
    GROUP BY user_id, activity_date
),
rolling_streak AS (
    SELECT 
        user_id,
        activity_date,
        daily_count,
        -- Check if there's activity in the last 7 days
        COUNT(*) OVER (
            PARTITION BY user_id 
            ORDER BY activity_date 
            RANGE BETWEEN INTERVAL '6' DAY PRECEDING AND CURRENT ROW
        ) as rolling_count
    FROM daily_activity
)
SELECT 
    user_id,
    activity_date,
    rolling_count
FROM rolling_streak
WHERE rolling_count = 7  -- Complete 7-day streak
ORDER BY user_id, activity_date;

``` 
 
### Key SQL Functions and Concepts 

#### Essential Functions: 

- ROW_NUMBER() - Creates sequential numbering
- LAG()/LEAD() - Access previous/next rows
- SUM() OVER() - Running totals for group identification
- COUNT() OVER() - Count within windows
- MIN()/MAX() - Find streak boundaries
     

#### Window Function Patterns: 

```sql

-- Basic consecutive grouping
ROW_NUMBER() OVER (ORDER BY date) - 
ROW_NUMBER() OVER (PARTITION BY condition ORDER BY date)

-- Running count for conditions
SUM(CASE WHEN condition THEN 1 ELSE 0 END) OVER (ORDER BY date)
``` 
 
#### Problem-Solving Framework 
Step 1: Define the Streak Condition 

    What constitutes a "streak element"?
    Any additional filtering criteria?
     

Step 2: Create Groups of Consecutive Records 

    Use the row number difference technique
    Handle partitioning by relevant dimensions
     

Step 3: Aggregate Within Groups 

    Count records per group
    Calculate start/end dates
    Apply additional metrics
     

Step 4: Find Maximum/Desired Result 

    Use MAX(), MIN(), or other aggregations
    Consider ties and edge cases
     

#### Common Variations 
Time-Based Streaks: 

```sql

-- Ensure actual date consecutiveness, not just record order
SELECT 
    *,
    DATE_SUB(date_column, INTERVAL 
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY date_column) DAY
    ) AS date_group
FROM table_name;
``` 
 
#### Value-Based Streaks: 
```sql

-- Streaks of increasing/decreasing values
SELECT 
    *,
    ROW_NUMBER() OVER (ORDER BY date) -
    ROW_NUMBER() OVER (ORDER BY date, value) AS trend_group
FROM table_name;

``` 
