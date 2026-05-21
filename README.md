# MLB Sample Data Analysis: Advanced SQL Query Techniques

An advanced SQL project leveraging sample Major League Baseball (MLB) datasets to solve complex data challenges, optimize query performance, and uncover deep statistical insights. 

## 📌 Project Overview
The goal of this repository is to demonstrate mastery of complex SQL concepts using sports data. By analyzing player performance, team statistics, and historical trends, the queries in this project move beyond simple `SELECT` and `WHERE` clauses into advanced data manipulation, analytical functions, and optimization strategies. This project was developed as part of a Udemy course focusing on advanced SQL querying techniques. The scripts from this repository come from the course's final project. The final project contained questions about the data meant to harness advanced SQL techniques covered in the lessons.

---

## 🛠️ Tech Stack & Database
*   **Language:** SQL (Structured Query Language)
*   **Database:** PostgreSQL
*   **Tools:** pgAdmin4
*   **Dataset:** MLB Sample Data (including player stats, team records, salaries, and game-by-game outcomes).

---

## 🚀 Advanced SQL Concepts Covered
This project serves as a portfolio for the following advanced database techniques:

*   **Window Functions:** Utilizing `LAG()`, `LEAD()`, `RANK()`, `DENSE_RANK()`, and `SUM() OVER()` for running totals, year-over-year trends, and player rankings.
*   **Common Table Expressions (CTEs):** Writing clean, modular queries and leveraging recursive CTEs for hierarchical data structures.
*   **Subqueries & Derived Tables:** Implementing correlated and non-correlated subqueries to isolate specific data points (e.g., players beating team averages).
*   **Conditional Aggregation:** Using `CASE WHEN` statements with `SUM` and `COUNT` to pivot data and create customized stat sheets.
*   **Data Normalization & Joins:** Combining multiple tables (Players, Teams, Salaries, Awards) using complex multi-way joins, self-joins, and anti-joins.
*   **Query Optimization:** Utilizing indexing strategies, execution plans, and efficient filtering to handle large volumes of sports data.
---
## 📊 Key Insights Summary
This project analyzes historical Major League Baseball (MLB) data to uncover trends across collegiate pipelines, team financial strategies, and player physical and career evolutions.

#### 1. Collegiate Talent Pipelines
* **Exponential Pipeline Growth:** Tracks the expanding footprint of amateur baseball by calculating the unique number of schools producing MLB talent across consecutive decades.
* **Top-Tier Producers:** Identifies the all-time top 5 powerhouse universities responsible for generating the highest volume of major league players.
* **Decadal Dominance:** Utilizes advanced window functions (ROW_NUMBER()) to extract a historical timeline of the top 3 talent-producing schools for every single decade.

#### 2. Team Spending & Financial Benchmarks
* **The 20% Financial Elite:** Segregates and ranks the top quintile (20%) of MLB teams based on their average annual payroll using NTILE(5) serialization.
* **Cumulative Payroll Trajectories:** Generates a continuous, historical rolling total of team expenditures over time to visualize financial scaling.
* **The Billion-Dollar Threshold:** pinpoints the exact structural year a franchise’s cumulative spending crossed the $1 Billion milestone, establishing a benchmark for modern sports monetization.

#### 3. Player Career Lifespans & Loyalty
* **Career Longevity Analytics:** Calculates exact player ages at debut versus retirement, ranking the roster from the longest historical careers to the shortest.
* **Franchise Loyalty Metrics:** Tracks player movement by comparing debut-season teams against final-season teams.
* **The "One-Club" Decennial Club:** Isolates an elite tier of historical players who bucked the trend of modern free agency—specifically filtering for individuals who spent over 10 years in the majors while starting and ending their careers with the exact same franchise.

#### 4. Player Demographics & Physical Evolution
* **Birthday Synchronicity:** Groups and aggregates modern players (born between 1980 and 1990) who share identical birthdays using string aggregation (string_agg).
* **Team Batting Profiles:** Generates a roster-by-roster breakdown displaying the exact percentage of right-handed, left-handed, and switch-hitters (bats = 'B') for every team.
* **Physical Evolution Across Eras:** Measures the decade-over-decade variance in average player height and weight at the time of debut. By using LAG() window functions, it highlights the physical scaling of the modern professional athlete over time.

---
## 🔍️ Sample Queries

### Team Roster Demographics: Batting Hand Dominance
This query analyzes team roster composition by calculating the percentage breakdown of player batting handedness (Right, Left, or Switch-hitters) across different teams.

```sql
SELECT s.teamid,
	   ROUND(SUM(CASE WHEN p.bats = 'R' THEN 1 ELSE 0 END)::NUMERIC / COUNT(s.playerid) * 100, 1) AS right_bat,
	   ROUND(SUM(CASE WHEN p.bats = 'L' THEN 1 ELSE 0 END)::NUMERIC / COUNT(s.playerid) * 100, 1) AS left_bat,
	   ROUND(SUM(CASE WHEN p.bats = 'B' THEN 1 ELSE 0 END)::NUMERIC / COUNT(s.playerid) * 100, 1) AS amb_bat
FROM salaries s
JOIN players p ON s.playerid = p.playerid
GROUP BY s.teamid
ORDER BY teamid;
```

### How It Works
* **Roster Joins:** Links the salaries table to the players table using playerid to pair roster assignments with player batting profiles (bats).
* **Proportion Logic:** Uses CASE WHEN to count players matching a specific batting stance ('R' for Right, 'L' for Left, 'B' for Both/Switch), divides by total team headcount (COUNT), and multiplies by 100.
* **Formatting:** Converts values to NUMERIC to avoid integer division truncation and rounds the final percentage to one decimal place.
* **Aggregation:** Groups and sorts the entire dataset by teamid to provide a clean, team-by-team breakdown.

### Key Data Insights
* **Lineup Balance:** Quantifies a team's vulnerability or advantage against specific pitching matchups (e.g., identifying teams heavily skewed toward right-handed hitters).
* **Switch-Hitting Depth:** Isolates the prevalence of ambidextrous hitters (amb_bat), highlighting roster flexibility.
* **Roster Strategy:** Exposes organizational philosophies regarding platoon advantages and trends in player acquisition.

---
### Player Career Longevity & Age Analytics
This query analyzes historical player data to calculate key career milestones: the age at which a player debuted, the age at which they played their final game, and their total career length. It features robust data cleaning to handle missing dates and strings, ensuring accurate interval calculations.

```sql
WITH dob AS (SELECT playerid, namegiven, CONCAT(birthyear::TEXT, '-', birthmonth::TEXT, '-', birthday::TEXT)::DATE AS birth_date,
					debut, finalgame
			FROM players
			WHERE birthday IS NOT NULL AND birthday > 0),
			
null_op  AS (SELECT playerid, namegiven, birth_date, COALESCE(debut, LAG(debut) OVER(), LEAD(debut) OVER()) AS debut,
										 COALESCE(finalgame, LAG(finalgame) OVER (), LEAD(finalgame) OVER()) AS finalgame
			FROM dob)
SELECT playerid, namegiven,
	   EXTRACT(YEAR FROM AGE(debut, birth_date)) AS debut_age,
	   EXTRACT(YEAR FROM AGE(finalgame, birth_date)) AS final_age,
	   COALESCE(EXTRACT(YEAR FROM AGE(finalgame, debut))) AS career_length
FROM null_op
ORDER BY career_length DESC;
```
### How It Works
* **Date Reconstruction (dob CTE):** Parses separate integer columns (birthyear, birthmonth, birthday) into a standardized SQL DATE format using CONCAT and typecasting (::TEXT), while filtering out invalid or missing data.
* **Missing Data Imputation (null_op CTE):** Uses a fallback mechanism via COALESCE paired with window functions (LAG and LEAD). If a player is missing a debut or finalgame date, the query intelligently borrows the date from the chronologically adjacent player record.
* **Age & Longevity Calculation (Main Select):** Uses PostgreSQL's AGE() and EXTRACT(YEAR FROM...) functions to calculate exact ages at major career milestones and ranks players by the longest overall careers (ORDER BY career_length DESC).

### Key Data Insights
* **Data Resiliency:** By implementing LAG and LEAD for imputation, the dataset minimizes gaps caused by missing historical tracking data.
* **Milestone Outliers:** Sorting by career_length allows immediate identification of historical anomalies, "iron-man" streaks, or data entry errors (e.g., negative career lengths or extreme ages).
* **Demographic Trends:** This structure sets up the data perfectly for downstream visualization, such as plotting the average debut age over different eras.
---

## 🎓 Acknowledgments & Course Credits
This project was built as a capstone application for advanced database management and analytical query design.

* **Course:** *SQL for Data Analysis: Advanced SQL Querying Techniques* on Udemy.
* **Instructor:** Alice Zhao
* **Dataset Source:** Based on the historical Lahman Baseball Database, adapted for specialized optimization and analytical learning tracks within the course curriculum.
