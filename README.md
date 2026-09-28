# DACH Data Job Market Analysis

## Overview

This project is part of my public [Data Science Project repository](https://github.com/fallerdavid98-ai/Data_Science_Project/tree/main) and analyzes the demand for technical skills in data-related job postings across the DACH region. For this analysis, the region includes **Germany, Austria, Switzerland, and Luxembourg**.

The project uses the job-posting dataset provided through [Luke Barousse's Python course](https://lukebarousse.com/python). The current completed analysis focuses on the skills requested for three major data roles: **Data Analyst, Data Engineer, and Data Scientist**.

Further analyses of skill trends, salaries, and optimal skills are planned and are explicitly marked as **Work in Progress (WIP)** below.

## Project Navigation

- [Repository home](https://github.com/fallerdavid98-ai/Data_Science_Project/tree/main)
- [Advanced Python and Pandas exercises](2_Advanced/)
- [Project notebooks](3_Project/)
- [Exploratory Data Analysis notebook](3_Project/1_EDA_Intro.ipynb)
- [DACH Skill Demand notebook](3_Project/2_Skill_Demand.ipynb)

## Project Status

| Analysis question | Status |
|---|---|
| 1. Which skills are most in demand for the selected data roles? | ✅ Completed |
| 2. How is demand for Data Analyst skills changing over time? | 🚧 WIP |
| 3. How well do data roles and skills pay? | 🚧 WIP |
| 4. Which skills offer the best combination of demand and salary? | 🚧 WIP |

## The Questions

This project is designed to answer the following questions:

1. Which skills are most in demand for Data Analysts, Data Engineers, and Data Scientists in the DACH region?
2. How are in-demand skills trending for Data Analysts in the DACH region?
3. How well do data jobs and individual skills pay in the DACH region?
4. Which skills are optimal to learn when balancing demand and salary?

## Tools Used

- **Python:** Core programming language for the analysis.
  - **Pandas:** Data cleaning, filtering, reshaping, grouping, and aggregation.
  - **Matplotlib:** Figure creation and detailed plot formatting.
  - **Seaborn:** Statistical data visualization.
- **Jupyter Notebook:** Interactive development and documentation of the analytical workflow.
- **Visual Studio Code:** Development environment used to work with the notebook and project files.
- **Git and GitHub:** Version control and project publication.

## Data Preparation and Cleanup

### Import and clean the dataset

The dataset is loaded and converted into a Pandas DataFrame. Date values are converted to a datetime data type, while the serialized skill lists are converted into Python lists.

```python
import ast
import pandas as pd
from datasets import load_dataset
import matplotlib.pyplot as plt
from matplotlib.ticker import StrMethodFormatter
import seaborn as sns

dataset = load_dataset("lukebarousse/data_jobs")
df = dataset["train"].to_pandas()

df["job_posted_date"] = pd.to_datetime(df["job_posted_date"])
df["job_skills"] = df["job_skills"].apply(
    lambda x: ast.literal_eval(x) if pd.notna(x) else x
)
```

### Filter the DACH region

The analysis is restricted to job postings from Germany, Austria, Switzerland, and Luxembourg.

```python
DACH_countries = ["Germany", "Austria", "Switzerland", "Luxembourg"]

df_DACH = df[df["job_country"].isin(DACH_countries)].copy()
```

### Prepare the skill data

Each job posting can contain multiple skills. The `explode()` operation creates one row per listed skill, allowing skill occurrences to be counted by job title.

```python
df_DACH_expl = df_DACH.explode("job_skills")

df_DACH_skillcount = (
    df_DACH_expl
    .groupby(["job_skills", "job_title_short"])
    .size()
    .reset_index(name="skill_count")
)

df_DACH_skillcount.sort_values(
    by="skill_count",
    ascending=False,
    inplace=True
)

job_titles = df_DACH_skillcount["job_title_short"].unique().tolist()
job_titles = sorted(job_titles[:3])
```

Related learning notebooks used to build these preparation steps:

- [Pandas Data Cleaning](2_Advanced/2_Pandas_Data_cleaning.ipynb)
- [Pandas Data Management](2_Advanced/3_Pandas_Data_Management.ipynb)
- [Pandas Merge DataFrames](2_Advanced/7_Pandas_Merge_DataFrames.ipynb)

## The Analysis

## 1. Which skills are most in demand for the selected data roles?

View the complete analysis and executable code in the [DACH Skill Demand notebook](3_Project/2_Skill_Demand.ipynb).

### Method

For every combination of job title and skill, the analysis counts how many job postings mention that skill. The count is then divided by the **total number of original job postings for the respective job title**.

The resulting percentage answers questions such as:

> What percentage of all DACH Data Engineer postings mention Python?

Because one posting can request several skills, the percentages of all skills within a role are not expected to sum to 100%.

```python
df_DACH_totaljobs = (
    df_DACH["job_title_short"]
    .value_counts(ascending=False)
    .reset_index(name="total_skill_count")
)

df_DACH_skilljob_merge = df_DACH_skillcount.merge(
    right=df_DACH_totaljobs,
    how="left",
    on="job_title_short"
)

df_DACH_skilljob_merge["skill_share"] = 100 * (
    df_DACH_skilljob_merge["skill_count"]
    / df_DACH_skilljob_merge["total_skill_count"]
)
```

> **Note:** Despite its name in the notebook, `total_skill_count` contains the number of original job postings per job title and is therefore the denominator for the posting-level percentage.

### Visualize the results

The final chart displays the five most frequently requested skills for each selected data role.

The subplot implementation builds on the techniques practiced in the [Subplots exercise notebook](2_Advanced/11_Exercise_Subplots.ipynb).

```python
fig, ax = plt.subplots(len(job_titles), 1)

sns.set_theme(style="ticks")

for i, title in enumerate(job_titles):
    df_DACH_skilljob_plot = df_DACH_skilljob_merge[
        df_DACH_skilljob_merge["job_title_short"] == title
    ].head(5)

    sns.barplot(
        data=df_DACH_skilljob_plot,
        x="skill_share",
        y="job_skills",
        ax=ax[i],
        hue="skill_count",
        palette="dark:b_r"
    )

    ax[i].set_title(title)
    ax[i].set_xlabel("")
    ax[i].set_ylabel("")
    ax[i].legend().set_visible(False)
    ax[i].set_xlim(0, 70)

    if i != len(job_titles) - 1:
        ax[i].set_xticks([])

    for j, value in enumerate(df_DACH_skilljob_plot["skill_share"]):
        ax[i].text(value + 0.5, j, f"{value:.0f}%", va="center")
        ax[i].xaxis.set_major_formatter(
            plt.FuncFormatter(lambda x, _: f"{int(x)}%")
        )

fig.suptitle(
    "Likelihood of Skills Requested in DACH job postings",
    fontsize=15
)
fig.tight_layout()
plt.show()
```

### Results

![Likelihood of Skills Requested in DACH Job Postings](images/Likelihood_of_Skills_Requested_in_DACH_Job_Postings.png)

*Percentage of DACH job postings that mention each of the five most frequently requested skills for Data Analysts, Data Engineers, and Data Scientists.*

### Insights

- **Data Analysts:** SQL is the leading skill and appears in **42%** of postings, followed by Python at **32%**. Excel, Tableau, and Power BI form a second group with demand between **18% and 21%**.
- **Data Engineers:** Python and SQL dominate at **53%** and **50%**. Cloud and distributed-processing skills are also prominent: Azure appears in **31%**, AWS in **23%**, and Spark in **20%** of postings.
- **Data Scientists:** Python is the clearest leading skill at **61%**. SQL follows at **34%**, while R remains important at **29%**. Azure and AWS appear less frequently, at **13%** and **11%**.
- **Across roles:** Python and SQL are the only skills represented among the top five for all three roles, demonstrating their broad relevance across the DACH data job market.

## 2. How are in-demand skills trending for Data Analysts?

> [!IMPORTANT]
> **Work in Progress:** The monthly trend analysis has not yet been implemented in the current DACH notebook. No chart or conclusion is reported at this stage.

Planned work:

- Filter DACH Data Analyst postings.
- Aggregate skill mentions by posting month.
- Normalize monthly skill counts by the number of monthly Data Analyst postings.
- Visualize the development of the leading skills over time.

Methodological starting point: [Trending Skills exercise notebook](2_Advanced/10_Exercise_Trending_Skills.ipynb).

## 3. How well do jobs and skills pay?

> [!IMPORTANT]
> **Work in Progress:** The DACH salary analysis is still open. No salary results from the U.S. reference project are transferred to this regional analysis.

Planned work:

- Examine salary availability and missing values in the DACH subset.
- Compare annual salary distributions across common data roles.
- Calculate median salaries associated with individual skills.
- Distinguish robust results from observations based on small samples.

Methodological starting point: [Histograms and Boxplots exercise notebook](2_Advanced/13_Exercise_Histograms_Boxplots.ipynb).

## 4. Which skills offer the best combination of demand and salary?

> [!IMPORTANT]
> **Work in Progress:** This analysis depends on the completion and validation of the DACH salary analysis.

Planned work:

- Combine skill-demand percentages with median salary estimates.
- Apply a minimum observation threshold to reduce small-sample distortion.
- Compare skills by demand, compensation, and technology category.
- Visualize the results in a demand-versus-salary scatter plot.

Methodological starting point: [Scatterplots exercise notebook](2_Advanced/12_Exercise_Scatterplots.ipynb).

## What I Learned So Far

- **Choosing the correct denominator matters:** Skill demand should be calculated against the number of original job postings, not against the number of rows produced by `explode()`.
- **Exploding list columns enables grouped analysis:** Converting each skill list into individual rows makes it possible to count and compare skill occurrences efficiently.
- **Percentages can overlap:** Since a single posting can request multiple skills, posting-level skill percentages do not need to sum to 100%.
- **Regional filtering changes the analytical context:** Findings for the DACH region should not be copied from a U.S.-focused analysis without recalculation.
- **Clear chart annotations improve readability:** Direct percentage labels make the comparison of skill demand easier across roles.

## Challenges

- Ensuring that the percentage denominator represents job postings rather than exploded skill rows.
- Handling postings with missing skill information without treating missing values as actual skills.
- Keeping the same visual scale across subplots while preserving readable labels.
- Separating completed findings from planned analyses so that WIP sections do not imply unsupported conclusions.

## Conclusion

The completed first stage shows that Python and SQL are central across the selected DACH data roles, while role-specific patterns remain visible: analyst roles place greater emphasis on reporting and visualization tools, engineering roles on cloud and distributed-processing technologies, and data-science roles on Python and R.

The project remains in progress. Trend, salary, and optimal-skill analyses will be added only after their DACH-specific calculations and visualizations have been completed and validated.
