# DACH Data Job Market Analysis

## Overview

This project is part of my public [Data Analysis Project repository](https://github.com/fallerdavid98-ai/Data_Analysis_Project/tree/main) and analyzes the demand for technical skills in data-related job postings across the DACH region. For this analysis, the region includes **Germany, Austria, Switzerland, and Luxembourg**.

The project uses the job-posting dataset provided through [Luke Barousse's Python course](https://lukebarousse.com/python). The completed analyses currently cover the skills requested for three major data roles—**Data Analyst, Data Engineer, and Data Scientist**—and the monthly development of the five most frequently mentioned skills across DACH data jobs.

Further analyses of salaries and optimal skills are planned and are explicitly marked as **Work in Progress (WIP)** below.

## Project Navigation

- [Repository home](https://github.com/fallerdavid98-ai/Data_Analysis_Project/tree/main)
- [Advanced Python and Pandas exercises](2_Advanced/)
- [Project notebooks](3_Project/)
- [Exploratory Data Analysis notebook](3_Project/1_EDA_Intro.ipynb)
- [DACH Skill Demand notebook](3_Project/2_Skill_Demand.ipynb)
- [DACH Skill Trends notebook](3_Project/3_Skills_Trend.ipynb)

## Project Status

| Analysis question | Status |
|---|---|
| 1. Which skills are most in demand for the selected data roles? | ✅ Completed |
| 2. How are the leading skills trending across DACH data jobs? | ✅ Completed |
| 3. How well do data roles and skills pay? | 🚧 WIP |
| 4. Which skills offer the best combination of demand and salary? | 🚧 WIP |

## The Questions

This project is designed to answer the following questions:

1. Which skills are most in demand for Data Analysts, Data Engineers, and Data Scientists in the DACH region?
2. How are the leading skills trending across DACH data jobs?
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
from matplotlib.ticker import StrMethodFormatter, PercentFormatter
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

## 2. How are the leading skills trending across DACH data jobs?

View the complete analysis and executable code in the [DACH Skill Trends notebook](3_Project/3_Skills_Trend.ipynb).

### Method

The monthly trend analysis uses all DACH job postings in the dataset rather than filtering for a single job title. First, every posting receives a numerical month value and its skill list is expanded into individual rows. A pivot table then counts how often each skill appears in each month.

```python
df_DACH["job_posted_month"] = df_DACH["job_posted_date"].dt.month

df_DACH_expl = df_DACH.explode(column="job_skills")

df_DACH_pivot = df_DACH_expl.pivot_table(
    index="job_posted_month",
    columns="job_skills",
    aggfunc="size",
    fill_value=0
)
```

The skills are ranked by their total number of mentions across the full year. The temporary `total` row is used only for sorting and is removed before calculating the monthly percentages. This ensures that the chart follows the same five leading skills throughout the year instead of changing the selection from month to month.

```python
df_DACH_pivot.loc["total"] = df_DACH_pivot.sum()

df_DACH_pivot = df_DACH_pivot[
    df_DACH_pivot.loc["total"].sort_values(ascending=False).index
]

df_DACH_pivot.drop(index="total", inplace=True)
```

Raw monthly skill counts are normalized by the total number of DACH job postings in the respective month. The resulting value therefore represents the percentage of that month's postings that mention a skill.

```python
DACH_countbymonth = df_DACH.groupby("job_posted_month").size()

df_DACH_perc = df_DACH_pivot.div(
    DACH_countbymonth / 100,
    axis=0
)
```

For example, a Python value of 50% means that Python appears in approximately half of all DACH postings published in that month. Since one posting can mention several skills, the percentages shown for different skills are not intended to add up to 100%.

### Format and visualize the results

The numerical month values are converted into abbreviated month names before the five leading skills are plotted as continuous monthly trends.

```python
df_DACH_perc.reset_index(names="jobs_by_month", inplace=True)

df_DACH_perc["job_posted_month"] = df_DACH_perc["jobs_by_month"].apply(
    lambda x: pd.to_datetime(x, format="%m").strftime("%b")
)

df_DACH_perc.set_index("job_posted_month", inplace=True)
df_DACH_perc.drop(columns="jobs_by_month", inplace=True)

df_DACH_plot = df_DACH_perc.iloc[:, :5]

sns.lineplot(data=df_DACH_plot, dashes=False, palette="tab10")
sns.set_theme(style="ticks")

plt.title("Top 5 Job Skills for DACH Data Jobs (by Month)")
plt.ylabel("Prominence in Job Postings")
plt.xlabel("2023")
plt.legend().remove()
sns.despine()

for i in range(5):
    plt.text(
        11.3,
        df_DACH_plot.iloc[-1, i],
        df_DACH_plot.columns[i]
    )

ax = plt.gca()
ax.yaxis.set_major_formatter(PercentFormatter(decimals=0))

plt.show()
```

The workflow builds on the monthly aggregation techniques practiced in the [Trending Skills exercise notebook](2_Advanced/10_Exercise_Trending_Skills.ipynb).

### Results

![Top 5 Job Skills for DACH Data Jobs by Month](images/Top_5_Job_Skills_for_DACH_Data_Jobs.png)

*Monthly share of DACH job postings mentioning each of the five most frequently requested skills across the full year.*

### Insights

- **Python and SQL remain dominant:** Both skills lead throughout the year by a wide margin. Python stays above SQL in every month.
- **Demand softens in the second half of the year:** Python peaks at roughly **52%** of monthly postings in May and ends the year at about **42%**. SQL follows a similar pattern, falling from a May peak of approximately **44%** to around **37%** in December.
- **Azure is comparatively stable:** Azure remains close to **18%** for much of the year before declining to roughly **15%** during the fourth quarter.
- **R fluctuates more strongly:** R reaches local highs of around **19%** in April and early summer, but its monthly share falls to approximately **13%** by December.
- **AWS also declines toward year-end:** AWS generally stays in the mid-teen range during the first eight months and finishes the year at roughly **12%**.
- **The annual leaders remain consistent:** Although their monthly prominence changes, Python, SQL, Azure, R, and AWS form the five most frequently mentioned skills across the complete DACH dataset for 2023.

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
- **Monthly counts need normalization:** Dividing monthly skill mentions by the number of postings in the same month makes periods with different posting volumes comparable.
- **A fixed skill selection improves trend comparisons:** Selecting the five leading skills by their annual totals keeps the plotted categories consistent across all twelve months.

## Challenges

- Ensuring that the percentage denominator represents job postings rather than exploded skill rows.
- Handling postings with missing skill information without treating missing values as actual skills.
- Keeping the same visual scale across subplots while preserving readable labels.
- Separating completed findings from planned analyses so that WIP sections do not imply unsupported conclusions.
- Distinguishing changes in raw posting volume from changes in the percentage of postings that request a particular skill.
- Placing direct labels at the end of the trend lines without reducing readability or confusing closely positioned series.

## Conclusion

The completed analyses show that Python and SQL are central across the selected DACH data roles, while role-specific patterns remain visible: analyst roles place greater emphasis on reporting and visualization tools, engineering roles on cloud and distributed-processing technologies, and data-science roles on Python and R.

The monthly trend analysis confirms that Python and SQL remain the leading skills throughout 2023, even though their share of monthly postings declines during the second half of the year. Azure is comparatively stable before a fourth-quarter decrease, while R and AWS show more fluctuation and finish the year below their earlier levels.

The project remains in progress. Salary and optimal-skill analyses will be added only after their DACH-specific calculations and visualizations have been completed and validated.
