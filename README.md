# Introduction  
📊 This project takes a deep dive into the *data analyst job market*, exploring 💰 top-paying positions, 🔥 the most in-demand skills, and 📈 the intersection where high demand meets high salary potential.

# Background  
The purpose of this project is to better understand the career landscape for data analysts. It aims to identify which skills are most valuable, which roles offer the best salaries, and how skill demand connects with pay trends.  

### Key Questions Explored Using SQL  
1. Which data analyst jobs pay the most?  
2. What skills are required for those top-paying roles?  
3. Which skills are most in demand for data analysts?  
4. Which skills are associated with higher average salaries?  
5. Which skills are both high-paying and in demand — the best to focus on learning?  

# Tools and Technologies  
For this analysis, I used the following tools:  
- *SQL:* For querying and analyzing the job posting data.  
- *SQL Server Management Studio (SSMS):* To execute and manage SQL queries.  
- *GitHub:* For version control, documentation, and sharing of queries and insights.  

# Analytical Approach  
Each SQL query in this project answers a specific question about the data analyst job market. Combined, they provide a complete picture of current trends, in-demand skills, and salary insights.

---

### 1. Highest Paying Data Analyst Jobs  

To identify the best-paying roles, I filtered job listings by *average annual salary* and *location*, focusing on remote opportunities.  

sql
SELECT TOP 10
    job_id,
    job_title,
    job_location,
    job_schedule_type,
    salary_year_avg,
    job_posted_date,
    name AS company_name
FROM job_postings_fact
LEFT JOIN company_dim 
    ON job_postings_fact.company_id = company_dim.company_id
WHERE job_title_short = 'Data Analyst' 
  AND job_location = 'Anywhere' 
  AND salary_year_avg IS NOT NULL
ORDER BY salary_year_avg DESC;


*Findings:*  
- Top 10 data analyst salaries ranged from *$184,000 to $650,000*.  
- High-paying employers included *SmartAsset, **Meta, and **AT&T*.  
- Job titles ranged from Data Analyst to Director of Analytics, showing diversity in seniority.  

---

### 2. Skills Required for High-Paying Jobs  

This step identifies the most common skills among the top-paying job postings by joining the job and skill tables.  

sql
WITH top_paying_jobs AS (
    SELECT TOP 10
        job_id,
        job_title,
        salary_year_avg,
        name AS company_name
    FROM job_postings_fact
    LEFT JOIN company_dim 
        ON job_postings_fact.company_id = company_dim.company_id
    WHERE job_title_short = 'Data Analyst' 
      AND job_location = 'Anywhere' 
      AND salary_year_avg IS NOT NULL
    ORDER BY salary_year_avg DESC
)
SELECT 
    top_paying_jobs.*,
    skills_dim.skills AS skills
FROM top_paying_jobs
INNER JOIN skills_job_dim 
    ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim 
    ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY top_paying_jobs.salary_year_avg DESC;


*Insights:*  
- *SQL* appeared in 8 out of 10 top-paying jobs.  
- *Python* followed closely, showing up 7 times.  
- *Tableau, **R, **Snowflake, **Pandas, and **Excel* were also featured in top listings.  

---

### 3. Most In-Demand Skills for Data Analysts  

To find which skills are most commonly requested, I calculated the frequency of each skill in job postings.  

sql
SELECT TOP 5
    skills_dim.skills AS skills,
    COUNT(skills_job_dim.job_id) AS demand_count
FROM job_postings_fact
INNER JOIN skills_job_dim 
    ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim 
    ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE job_title_short = 'Data Analyst' 
  AND job_work_from_home = 1
GROUP BY skills_dim.skills
ORDER BY demand_count DESC;


*Results:*  

| Skill | Demand Count |
|--------|---------------|
| SQL | 7291 |
| Excel | 4611 |
| Python | 4330 |
| Tableau | 3745 |
| Power BI | 2609 |

*Summary:*  
- *SQL* and *Excel* are the backbone of most analytics jobs.  
- *Python, **Tableau, and **Power BI* highlight growing demand for programming and visualization.  

---

### 4. Skills Linked to Higher Salaries  

This query calculates the average salary associated with each skill to identify those most correlated with higher pay.  

sql
SELECT 
    skills,
    ROUND(AVG(salary_year_avg), 0) AS avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim 
    ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim 
    ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE job_title_short = 'Data Analyst'
  AND salary_year_avg IS NOT NULL
  AND job_work_from_home = True 
GROUP BY skills
ORDER BY avg_salary DESC
LIMIT 25;


*Results:*  

| Skill | Average Salary ($) |
|--------|-------------------:|
| PySpark | 208,172 |
| Bitbucket | 189,155 |
| Couchbase | 160,515 |
| Watson | 160,515 |
| DataRobot | 155,486 |
| GitLab | 154,500 |
| Swift | 153,750 |
| Jupyter | 152,777 |
| Pandas | 151,821 |
| Elasticsearch | 145,000 |

*Observations:*  
- High-paying skills include *PySpark, **DataRobot, and **Pandas*, reflecting strong demand for data engineering and ML expertise.  
- *Cloud tools* like *Databricks* and *Elasticsearch* also command higher salaries.  
- Development tools such as *GitLab* and *Airflow* are valued for automation and workflow management.  

---

### 5. Optimal Skills to Learn (High Demand + High Salary)  

This query combines demand and salary data to find the most valuable skills that balance market demand with compensation.  

sql
WITH skills_demand AS (
    SELECT
        s.skill_id,
        s.skills AS skills,
        COUNT(sj.job_id) AS demand_count
    FROM job_postings_fact AS j
    INNER JOIN skills_job_dim AS sj 
        ON j.job_id = sj.job_id
    INNER JOIN skills_dim AS s 
        ON sj.skill_id = s.skill_id
    WHERE j.job_title_short = 'Data Analyst'
      AND j.salary_year_avg IS NOT NULL
      AND j.job_work_from_home = 'True' 
    GROUP BY s.skill_id, s.skills
), 
average_salary AS (
    SELECT 
        sj.skill_id,
        ROUND(AVG(TRY_CAST(j.salary_year_avg AS FLOAT)), 0) AS avg_salary
    FROM job_postings_fact AS j
    INNER JOIN skills_job_dim AS sj 
        ON j.job_id = sj.job_id
    INNER JOIN skills_dim AS s 
        ON sj.skill_id = s.skill_id
    WHERE j.job_title_short = 'Data Analyst'
      AND j.salary_year_avg IS NOT NULL
      AND j.job_work_from_home = 'True'    
    GROUP BY sj.skill_id
)
SELECT TOP 25
    sd.skill_id,
    sd.skills,
    sd.demand_count,
    a.avg_salary
FROM skills_demand AS sd
INNER JOIN average_salary AS a 
    ON sd.skill_id = a.skill_id
WHERE sd.demand_count > 10
ORDER BY a.avg_salary DESC, sd.demand_count DESC;


*Results:*  

| Skill | Demand Count | Average Salary ($) |
|--------|---------------|-------------------:|
| Go | 27 | 115,320 |
| Confluence | 11 | 114,210 |
| Hadoop | 22 | 113,193 |
| Snowflake | 37 | 112,948 |
| Azure | 34 | 111,225 |
| BigQuery | 13 | 109,654 |
| AWS | 32 | 108,317 |
| Java | 17 | 106,906 |
| SSIS | 12 | 106,683 |
| Jira | 20 | 104,918 |

*Highlights:*  
- *Python* and *R* continue to be essential and well-compensated.  
- *Cloud platforms* like *AWS, **Azure, and **Snowflake* are increasingly important.  
- *Visualization tools* such as *Tableau* and *Looker* are critical for data storytelling.  
- *Database knowledge* (SQL Server, Oracle, NoSQL) remains fundamental.  

---

# Key Learnings  

Working on this project significantly improved my SQL and analytical abilities:  
- *🧩 Query Design:* Learned to build complex queries using joins, subqueries, and CTEs.  
- *📊 Aggregation:* Strengthened understanding of GROUP BY, COUNT(), and AVG().  
- *💡 Problem Solving:* Improved translating analytical questions into structured SQL solutions.  

---

# Conclusions  

### Major Insights  
1. *Top-Paying Roles:* Remote data analyst roles can pay as high as *$650,000*, showing strong salary potential.  
2. *Core Skill:* *SQL* remains the most in-demand and valuable skill for analysts.  
3. *High-Demand Skills:* SQL, Excel, Python, Tableau, and Power BI dominate job listings.  
4. *Top-Earning Specialties:* Niche skills such as Solidity and SVN lead to premium pay.  
5. *Best Investment Skill:* SQL stands out as the top skill for maximizing both employability and salary.  

### Final Thoughts  
This project provided a deeper understanding of the analytics job market and enhanced my ability to extract actionable insights from real-world data.  
It highlights the continued importance of *SQL, **Python, and **cloud technologies* in shaping the future of data analytics careers.
