# 🏏 IPL Cricket Analysis (2008–2025)
### End-to-End Data Analysis using SQL (SQLite) + Python EDA | 17 Seasons | 283,678 Ball-by-Ball Records

---

## 📌 Project Overview

This project performs a complete data analysis of the **Indian Premier League (IPL)** from its first season in 2008 through the **2025 season**, in which **Royal Challengers Bengaluru claimed their maiden IPL title** by defeating Punjab Kings in the final.

The analysis covers **17 seasons**, **1,169 matches**, and **283,678 ball-by-ball records** — exploring team dominance, player performance, match patterns, and the impact of toss decisions on match outcomes.

---

## 🎯 Objectives

- Build a complete data pipeline from raw CSV → data cleaning → SQLite database → SQL queries → EDA visualisations
- Answer 8 key analytical questions about IPL history using SQL
- Visualise patterns and insights using Matplotlib and Seaborn
- Highlight RCB's historic 2025 title-winning campaign

---

## 🗂️ Project Structure

```
ipl_analysis/
│
├── notebooks/
│   └── ipl_eda.ipynb              # Main Jupyter notebook
│
├── sql/
│   ├── q1_team_wins.sql
│   ├── q2_toss_impact.sql
│   ├── q3_top_scorers.sql
│   ├── q4_top_wickets.sql
│   ├── q5_runs_per_phase.sql
│   ├── q6_potm_awards.sql
│   ├── q7_top_venues.sql
│   └── q8_rcb_2025.sql
│
├── README.md
└── .gitignore
```

> **Note:** Dataset is not included in this repository due to file size. Download it from [Kaggle](https://www.kaggle.com/datasets/chaitu20/ipl-dataset2008-2025) and place it in a `data/` folder.

---

## 📦 Dataset

| Property | Details |
|----------|---------|
| **Source** | [Kaggle — IPL Dataset 2008–2025](https://www.kaggle.com/datasets/chaitu20/ipl-dataset2008-2025) |
| **Type** | Ball-by-ball data (single combined CSV) |
| **Raw columns** | 65 |
| **Columns used** | 22 (after selection) |
| **Total rows** | 283,678 |
| **Seasons covered** | 2008 to 2025 (17 seasons) |
| **Total matches** | 1,169 |

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|------|---------|
| **Python 3.10** | Core programming language |
| **Pandas** | Data loading, cleaning, manipulation |
| **SQLite3** | Database creation and SQL querying |
| **Matplotlib** | Chart creation and customisation |
| **Seaborn** | Chart styling and color palettes |
| **Jupyter Notebook** | Interactive development environment |
| **VSCode** | Code editor with SQLite Viewer extension |

---

## 🔧 Data Cleaning Steps

The raw dataset had 65 columns and required significant cleaning before analysis:

1. **Column selection** — Reduced from 65 → 22 relevant columns
2. **Null value handling**
   - `wicket_kind` → filled nulls with `'no_wicket'` (expected — most balls have no dismissal)
   - `player_out` → filled nulls with `'not_out'`
   - `result_type` → filled nulls with `'normal'`
   - `win_outcome` → filled nulls with `'no_result'` (abandoned matches)
3. **Duplicate removal** — Dropped 636 duplicate rows
4. **Team name standardisation** — Fixed historical name changes across all 4 team columns:
   - `Delhi Daredevils` → `Delhi Capitals`
   - `Kings XI Punjab` → `Punjab Kings`
   - `Deccan Chargers` → `Sunrisers Hyderabad`
   - `Royal Challengers Bangalore` → `Royal Challengers Bengaluru`
   - `Rising Pune Supergiants` → `Rising Pune Supergiant`
5. **Season format fix** — Converted `2007/08` → `2008`, `2009/10` → `2010` etc.
6. **Season splitting** — Separated merged `2020/21` data back into correct years (2020 UAE season + 2021)
7. **Incomplete data removal** — Dropped 24 matches from early IPL 2026 that were incorrectly included

**Final dataset: 283,678 rows × 22 columns, fully clean**

---

## 🗄️ SQLite Database

The cleaned DataFrame was loaded into a SQLite database (`ipl_2025.db`) using Python's `sqlite3` library:

```python
import sqlite3
conn = sqlite3.connect('ipl_2025.db')
df.to_sql('ipl', conn, if_exists='replace', index=False)
```

All 8 SQL queries were run using `pd.read_sql()` — bridging SQL and Python in one workflow.

---

## 📊 SQL Queries & Key Findings

### Q1 — Most Wins by Team (All Time)
```sql
SELECT match_won_by AS team, COUNT(DISTINCT match_id) AS wins
FROM ipl
WHERE match_won_by != 'no_result' AND innings = 1
GROUP BY match_won_by ORDER BY wins DESC
```
**Finding:** Mumbai Indians lead all-time with **151 wins**, followed by CSK (142) and KKR (135).

---

### Q2 — Does Winning the Toss Help?
```sql
SELECT toss_decision,
       COUNT(*) AS total_matches,
       ROUND(100.0 * SUM(CASE WHEN toss_winner = match_won_by THEN 1 ELSE 0 END)
             / COUNT(*), 2) AS win_pct
FROM (SELECT DISTINCT match_id, toss_decision, toss_winner, match_won_by
      FROM ipl WHERE match_won_by != 'no_result' AND innings = 1)
GROUP BY toss_decision
```
**Finding:** Teams choosing to **field win 53%** of the time vs **45% when choosing to bat** — chasing is the dominant strategy in T20 cricket.

---

### Q3 — Top 15 Run Scorers (Career)
```sql
SELECT batter, SUM(runs_batter) AS total_runs,
       COUNT(*) AS balls_faced,
       ROUND(SUM(runs_batter) * 100.0 / COUNT(*), 2) AS strike_rate
FROM ipl
GROUP BY batter ORDER BY total_runs DESC LIMIT 15
```
**Finding:** **Virat Kohli** leads all-time with **8,671 runs** — 1,623 more than second-placed Rohit Sharma (7,048).

---

### Q4 — Top 15 Wicket Takers (Career)
```sql
SELECT bowler, COUNT(*) AS wickets
FROM ipl
WHERE wicket_kind NOT IN ('no_wicket','run out','retired hurt','obstructing the field')
GROUP BY bowler ORDER BY wickets DESC LIMIT 15
```
**Finding:** **Yuzvendra Chahal** is the all-time leading wicket taker with **221 wickets**. 8 of the top 15 are spinners.

---

### Q5 — Average Runs Per Over Phase
```sql
SELECT
  CASE
    WHEN over BETWEEN 0 AND 5  THEN '1. Powerplay (0-5)'
    WHEN over BETWEEN 6 AND 14 THEN '2. Middle (6-14)'
    ELSE '3. Death (15-19)'
  END AS phase,
  ROUND(AVG(runs_total) * 6, 2) AS avg_runs_per_over
FROM ipl GROUP BY phase ORDER BY phase
```

| Phase | Avg Runs/Over |
|-------|--------------|
| Powerplay (0–5) | 7.66 |
| Middle (6–14) | 7.63 |
| Death (15–19) | **9.52** |

**Finding:** Death overs average **9.52 runs/over** — 24% higher than middle overs. Death bowling is the most decisive phase.

---

### Q6 — Most Player of the Match Awards
```sql
SELECT player_of_match, COUNT(DISTINCT match_id) AS potm_awards
FROM ipl WHERE innings = 1
GROUP BY player_of_match ORDER BY potm_awards DESC LIMIT 10
```
**Finding:** **AB de Villiers** leads with **25 POTM awards** — roughly 1 in every 5 matches he played.

---

### Q7 — Top Venues
```sql
SELECT venue, COUNT(DISTINCT match_id) AS matches_hosted
FROM ipl GROUP BY venue ORDER BY matches_hosted DESC LIMIT 10
```
**Finding:** **Eden Gardens** (77 matches) edges out Wankhede Stadium (73) as the most-used IPL venue.

---

### Q8 — RCB 2025 Season Journey 🏆
```sql
SELECT DISTINCT match_id, date, batting_team, bowling_team,
                match_won_by, win_outcome, stage
FROM ipl
WHERE season = 2025
  AND (batting_team = 'Royal Challengers Bengaluru'
       OR bowling_team = 'Royal Challengers Bengaluru')
  AND innings = 1
ORDER BY date
```
**Finding:** RCB won **10 of 14 league matches**, then beat Punjab Kings in both **Qualifier 1** and the **Final** (by 6 runs) to claim their maiden IPL title after 18 years.

---

## 📈 EDA Visualisations

| Chart | Type | Description |
|-------|------|-------------|
| Chart 1 | Horizontal Bar | All-time wins by team |
| Chart 2 | Side-by-side Bar | Toss decision impact on win % |
| Chart 3 | Horizontal Bar | Top 15 career run scorers |
| Chart 4 | Horizontal Bar | Top 15 career wicket takers |
| Chart 5 | Scatter Plot | Runs vs Strike Rate — top 15 batters |
| Chart 6 | Bar Chart | Most Player of the Match awards |
| Chart 7 | Bar Chart | Average runs per over phase |
| Chart 8 | Custom Bar | RCB 2025 match-by-match journey |

---

## 🔑 Key Insights Summary

1. **Mumbai Indians** are the most dominant IPL franchise with 151 wins across 17 seasons
2. **Toss strategy matters** — teams that choose to field after winning the toss win 8% more often than those who bat
3. **Virat Kohli** is in a class of his own — 8,671 runs, over 1,600 ahead of the next player
4. **Spin dominates** — 8 of the top 15 wicket takers are spinners, reflecting Indian pitch conditions
5. **Death overs are decisive** — averaging 9.52 runs/over, teams with strong death bowlers have a massive advantage
6. **RCB 2025** was their best-ever campaign — 10 league wins, dominant Qualifier 1, and a nail-biting 6-run final win

---



## 📚 Skills Demonstrated

- **Data wrangling** — handling nulls, duplicates, type conversions, column standardisation
- **SQL querying** — GROUP BY, COUNT DISTINCT, CASE WHEN, subqueries, aggregate functions
- **SQLite integration** — creating and querying a database entirely from Python
- **Exploratory Data Analysis** — pattern discovery across 17 seasons of data
- **Data visualisation** — 8 publication-quality charts with annotations and insights
- **Problem solving** — resolved multiple dataset-specific issues (merged seasons, team name changes, mislabelled entries)

---

## 👤 Author

**Akshay**
B.Tech — Electronics and Communication Engineering
Aspiring Data Analyst | Data Science Enthusiast

---

