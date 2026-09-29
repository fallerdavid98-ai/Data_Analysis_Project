# DACH Data Job Market Analysis

## Overview

This project is part of my public [Data Analysis Project repository](https://github.com/fallerdavid98-ai/Data_Analysis_Project/tree/main) and analyzes the demand for technical skills in data-related job postings across the DACH region. For this analysis, the region includes **Germany, Austria, Switzerland, and Luxembourg**.

The project uses the job-posting dataset provided through [Luke Barousse's Python course](https://lukebarousse.com/python). It begins with an exploratory overview of the DACH job market and then analyzes role-specific skill demand, monthly skill trends, salary distributions, and the relationship between skill prominence and median salary.

All five project notebooks are now included in the documented analytical workflow.

## Project Navigation

- [Repository home](https://github.com/fallerdavid98-ai/Data_Analysis_Project/tree/main)
- [Advanced Python and Pandas exercises](2_Advanced/)
- [Project notebooks](3_Project/)
- [Exploratory Data Analysis notebook](3_Project/1_EDA_Intro.ipynb)
- [DACH Skill Demand notebook](3_Project/2_Skill_Demand.ipynb)
- [DACH Skill Trends notebook](3_Project/3_Skills_Trend.ipynb)
- [DACH Salary Analysis notebook](3_Project/4_Salary_Analysis.ipynb)
- [DACH Optimal Skills notebook](3_Project/5_Optimal_Skills.ipynb)

## Project Status

| Analysis question | Status |
|---|---|
| 0. What does the DACH data-job landscape look like? | ✅ Completed |
| 1. Which skills are most in demand for the selected data roles? | ✅ Completed |
| 2. How are the leading skills trending across DACH data jobs? | ✅ Completed |
| 3. How well do data roles and skills pay? | ✅ Completed |
| 4. Which skills offer the best combination of prominence and salary? | ✅ Completed |

## The Questions

This project is designed to answer the following questions:

1. Which skills are most in demand for Data Analysts, Data Engineers, and Data Scientists in the DACH region?
2. How are the leading skills trending across DACH data jobs?
3. How well do data jobs and individual skills pay in the DACH region?
4. Which skills offer the strongest balance between prominence and median salary?

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

## Exploratory Data Analysis

View the complete introductory analysis and executable code in the [DACH Exploratory Data Analysis notebook](3_Project/1_EDA_Intro.ipynb).

The exploratory step establishes the regional context before the project moves into skill and salary analysis. It examines where DACH data jobs are advertised, which employment-related attributes are recorded, and which companies appear most frequently in the dataset.

### Leading job-location labels

The postings are grouped by `job_location`, counted, and sorted to identify the 15 most frequent location labels.

```python
df_DACH_plot = (
    df_DACH
    .groupby("job_location")
    .size()
    .sort_values(ascending=False)
    .head(15)
    .to_frame(name="count")
)
```

![Number of Data Jobs in DACH Countries](images/Number_of_Data_Jobs_in_DACH_Countries.png)

*The 15 most frequently recorded location labels for DACH data-job postings.*

#### Insights

- **Berlin and Vienna lead the ranking:** Berlin records the highest number of postings, followed by Vienna. Munich and Zürich are also among the strongest city-level locations.
- **Austria appears both as a country and through individual cities:** Country-level labels such as `Austria`, `Germany`, and `Switzerland` coexist with city-level entries.
- **Remote or non-specific locations form a separate category:** `Anywhere` is one of the most frequent labels, but it should not be interpreted as a physical DACH location.
- **The ranking mixes different geographical grains:** Cities, countries, and remote-location labels are compared in the same chart. It is therefore a useful overview of the raw location field, not a clean city-only market ranking.

### Work-from-home, degree, and health-insurance indicators

Three Boolean fields provide an initial view of selected characteristics recorded for DACH postings:

```python
eda_columns = [
    "job_work_from_home",
    "job_no_degree_mention",
    "job_health_insurance"
]

for column in eda_columns:
    print(df_DACH[column].value_counts(normalize=True) * 100)
```

![Remote Work Degree Requirement and Health Insurance Indicators](images/Remote_Work_Degree_Requirement_Health_Insurance_Plot.png)

*Shares of the Boolean work-from-home, no-degree-mention, and health-insurance fields in the DACH subset.*

#### Insights

- **Explicit work-from-home offers are uncommon:** Only **4.1%** of postings are marked `True` for `job_work_from_home`. This reflects the dataset flag and may not capture every hybrid-working arrangement described in the original vacancy text.
- **No degree is mentioned in a substantial minority of postings:** `job_no_degree_mention` is `True` for **41.7%** of the DACH subset. Because the column describes the absence of a degree mention, `True` must not be interpreted as a degree requirement.
- **The health-insurance field is not informative for this regional analysis:** Every DACH observation is marked `False`. This should not be read as evidence that DACH employers provide no health coverage; the field is atypical for the European context and likely reflects the source schema rather than actual benefit availability.

### Companies with the most postings

Before counting employers, different Deutsche-Bahn labels are consolidated into one company name. The 15 companies with the most DACH postings are then selected.

```python
df_DACH.loc[
    df_DACH["company_name"].str.contains(
        "Deutsche Bahn",
        na=False,
        regex=False
    ),
    "company_name"
] = "Deutsche Bahn"

df_DACH_plot2 = (
    df_DACH
    .groupby("company_name")
    .size()
    .sort_values(ascending=False)
    .head(15)
    .to_frame(name="count")
)
```

![Number of Data Jobs in DACH Countries by Company](images/Number_of_Data_Jobs_in_DACH_Countries_by_Company.png)

*The 15 company labels with the most data-job postings in the DACH subset.*

#### Insights

- **Deutsche Bahn is the leading employer in the dataset:** After consolidating its naming variants, it records the highest number of DACH postings by a clear margin.
- **Turing ranks second:** Workwise GmbH, Michael Page, Hays, GITR, and ROCKEN form the next group of frequently represented companies.
- **Company-name standardization affects the ranking:** `Bosch Group` and `Bosch Gruppe` still appear as separate labels. A broader entity-resolution step could combine further aliases and change individual positions in the employer ranking.

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

View the complete analysis and executable code in the [DACH Salary Analysis notebook](3_Project/4_Salary_Analysis.ipynb).

The salary analysis consists of two parts. The first compares the salary distributions of the six most frequently represented DACH data roles in the salary subset. The second contrasts the skills with the highest median salaries against the skills mentioned most frequently in salary-reported postings.

> [!CAUTION]
> **Limited salary coverage:** Both parts use only postings that contain a value for `salary_year_avg`. Salary information is available for a comparatively small number of DACH postings, so the results should be interpreted as descriptive findings for the available sample rather than precise estimates for the entire DACH job market.

### 3.1 Salary distributions of the six leading data roles

#### Method

The DACH dataset is filtered to postings with a reported annual salary. The six most frequent job titles are then selected from this reduced subset and ordered by median salary.

```python
DACH_countries = ["Germany", "Austria", "Switzerland", "Luxembourg"]

df_DACH = (
    df[df["job_country"].isin(DACH_countries)]
    .dropna(subset=["salary_year_avg"])
    .copy()
)

job_titles = (
    df_DACH["job_title_short"]
    .value_counts()
    .index[:6]
    .unique()
    .tolist()
)

df_DACH_top6 = df_DACH[
    df_DACH["job_title_short"].isin(job_titles)
]

jobs_orderedby_median = (
    df_DACH_top6
    .groupby("job_title_short")["salary_year_avg"]
    .median()
    .sort_values(ascending=False)
    .index
    .to_list()
)
```

Because the six roles are selected only after missing salaries have been removed, “leading” refers to the most frequent roles **among postings with salary information**, not necessarily the six most frequent roles in the complete DACH dataset.

For each boxplot, the number of observations, first quartile, median, and third quartile are calculated as an additional numerical summary.

```python
salary_statistics = (
    df_DACH_top6
    .groupby("job_title_short")["salary_year_avg"]
    .agg(
        count="count",
        q1=lambda x: x.quantile(0.25),
        median="median",
        q3=lambda x: x.quantile(0.75)
    )
)

salary_statistics.sort_values("median", ascending=False)
```

The salary distributions are displayed as horizontal boxplots and ordered from the highest to the lowest median.

```python
sns.boxplot(
    data=df_DACH_top6,
    x="salary_year_avg",
    y="job_title_short",
    order=jobs_orderedby_median
)

sns.set_theme(style="ticks")

plt.title("Salary Distributions in DACH Countries (in Order of Median Salary)")
plt.xlabel("Yearly Salary (USD)")
plt.ylabel("")
plt.xlim(0, 250_000)
plt.gca().xaxis.set_major_formatter(
    plt.FuncFormatter(lambda x, _: f"${int(x / 1000)}K")
)

plt.show()
```

The boxplots build on the distribution techniques practiced in the [Histograms and Boxplots exercise notebook](2_Advanced/13_Exercise_Histograms_Boxplots.ipynb).

#### Results

![Salary Distributions in DACH Countries](images/Salary_Distributions_in_DACH_Countries.png)

*Annual salary distributions for the six most frequent job titles among DACH postings with reported salary information, ordered by median salary.*

The following table supplements the boxplots with the underlying sample sizes and quartile values:

![Salary Distribution Statistics for DACH Countries](images/Addition_to_Salary_Distribution_in_DACH_Countries.png)

*Count of salary observations, first quartile, median, and third quartile for each displayed job title.*

#### Insights

- **Senior Data Scientists show the highest sample median:** Their median annual salary is **$157,500**, based on **26** salary observations.
- **Data Engineers and Senior Data Engineers share the same median:** Both groups have a median salary of **$147,500**, despite representing different seniority levels. This may partly reflect the limited sample size, repeated salary values, or differences in which employers disclose salaries.
- **Data Scientists occupy the middle of the ranking:** Their median is **$120,814**, based on **66** observations.
- **Data Analysts and Machine Learning Engineers have the lowest medians in this sample:** Their respective medians are **$90,550** and **$89,100**. The Machine Learning Engineer distribution is especially broad, with the median equal to the first quartile and the third quartile reaching **$166,000**.
- **Some median lines appear to be missing:** They are still present but overlap a box boundary. The median equals the third quartile for Senior Data Scientists, Data Engineers, and Senior Data Engineers, while the Machine Learning Engineer median equals the first quartile.
- **The ranking is not fully robust:** The displayed role-level sample sizes range only from **26 to 66** observations. The identical medians for Data Engineers and Senior Data Engineers should therefore not be interpreted as proof that both roles generally offer the same compensation across the DACH market.

### 3.2 Median salary of top-paying and frequently requested skills

#### Method

The skill lists from salary-reported postings are expanded into individual rows. Two separate rankings are then created:

1. the ten skills with the highest median salary;
2. the ten most frequently mentioned skills, reordered by their median salary for the visualization.

```python
df_DACH_expl = df_DACH.explode(column="job_skills").copy()

df_DACH_skillsbymedian = (
    df_DACH_expl
    .groupby("job_skills")["salary_year_avg"]
    .agg(["count", "median"])
    .sort_values(by="median", ascending=False)
    .head(10)
)

df_DACH_skillsbyprom = (
    df_DACH_expl
    .groupby("job_skills")["salary_year_avg"]
    .agg(["count", "median"])
    .sort_values(by="count", ascending=False)
    .head(10)
    .sort_values(by="median", ascending=False)
)
```

The `count` value in this section represents skill mentions only within postings that report an annual salary. The calculation therefore does not measure total skill demand across every DACH posting.

Both rankings are plotted on the same salary scale to make their median values directly comparable.

```python
fig, ax = plt.subplots(2, 1)
sns.set_theme(style="ticks")

sns.barplot(
    data=df_DACH_skillsbymedian,
    x="median",
    y=df_DACH_skillsbymedian.index,
    hue="median",
    ax=ax[0],
    palette="dark:b_r"
)

ax[0].legend().remove()
ax[0].set_title("Top 10 Best Paid Skills for Data Jobs in DACH")
ax[0].set_xlabel("")
ax[0].set_ylabel("")
ax[0].set_xlim(0, 250_000)

sns.barplot(
    data=df_DACH_skillsbyprom,
    x="median",
    y=df_DACH_skillsbyprom.index,
    hue="median",
    ax=ax[1],
    palette="dark:b_r"
)

ax[1].legend().remove()
ax[1].set_title("Top 10 Most In-Demand Skills for Data Jobs in DACH")
ax[1].set_xlabel("Median Salary (USD)")
ax[1].set_ylabel("")
ax[1].set_xlim(0, 250_000)

for axis in ax:
    axis.xaxis.set_major_formatter(
        plt.FuncFormatter(lambda x, _: f"${int(x / 1000)}K")
    )

fig.tight_layout()
plt.show()
```

#### Results

![Median Salary versus Skill Demand for Data Jobs in DACH](images/Median_Salary_vs_Skill_Demand_for_Data_Jobs_in_DACH.png)

*Comparison between the skills with the highest observed median salaries and the most frequently mentioned skills within the DACH salary subset.*

#### Insights

- **Rust has the highest observed median salary:** It stands clearly above the other skills in the salary-ranked group, while FastAPI follows in second place.
- **Several specialized skills share similarly high medians:** Firebase, Kotlin, Keras, NLTK, Perl, NoSQL, Vue, and TensorFlow cluster around a median salary of approximately **$157,500**.
- **AWS and Spark lead among the frequently mentioned skills:** Both combine comparatively high medians with strong representation in the salary subset. Git follows, while Python, SQL, and Azure form a middle group with similar median salaries.
- **Frequently requested skills produce the more market-relevant comparison:** Their larger number of mentions generally makes them more informative than niche skills that appear at the top of the salary ranking on the basis of very few observations.
- **The highest-paying-skill ranking is particularly sensitive to small samples:** No minimum `count` threshold is applied before selecting the ten highest medians. A skill can therefore enter the ranking because of only a small number of salary-reported postings.
- **These results are descriptive, not causal:** The analysis does not show that learning a particular skill causes a higher salary. Seniority, job title, employer, country, and the limited availability of salary information may all influence the observed medians.

## 4. Which skills offer the best combination of prominence and salary?

View the complete analysis and executable code in the [DACH Optimal Skills notebook](3_Project/5_Optimal_Skills.ipynb).

### Method

The final analysis combines two measures for every skill found in DACH postings with a reported annual salary:

- `skill_perc`: the percentage of salary-reported postings that mention the skill;
- `median_salary`: the median annual salary associated with those postings.

```python
df_DACH = (
    df[df["job_country"].isin(DACH_countries)]
    .dropna(subset=["salary_year_avg"])
    .copy()
)

df_DACH_expl = df_DACH.explode(column="job_skills")

df_DACH_skillset = (
    df_DACH_expl
    .groupby("job_skills")
    .agg(
        skill_count=("job_skills", "count"),
        median_salary=("salary_year_avg", "median")
    )
    .sort_values(by="skill_count", ascending=False)
)

total_job_count = len(df_DACH)

df_DACH_skillset["skill_perc"] = (
    df_DACH_skillset["skill_count"]
    / total_job_count
) * 100
```

Only skills mentioned in at least **8%** of the salary-reported DACH postings are retained. This removes very rare skills from the comparison and reduces the small-sample distortion observed in the unrestricted salary ranking.

```python
skill_prom = 8

df_DACH_skills_filtered = df_DACH_skillset[
    df_DACH_skillset["skill_perc"] >= skill_prom
]
```

The remaining skills are assigned to technology categories—such as programming, libraries, cloud, analyst tools, and other—using the dataset's `job_type_skills` dictionaries. The resulting DataFrame is visualized as a scatter plot.

```python
sns.scatterplot(
    data=df_DACH_techmerge,
    x="skill_perc",
    y="median_salary",
    hue="technology"
)

plt.xlabel("Prominence of Job Postings")
plt.ylabel("Median Yearly Salary")
plt.title("Salary vs. Prominence of Job Postings for Top Skills")

ax.yaxis.set_major_formatter(
    plt.FuncFormatter(lambda x, _: f"${int(x / 1000)}K")
)
ax.xaxis.set_major_formatter(StrMethodFormatter("{x:,.0f}%"))

plt.show()
```

The visualization builds on the techniques practiced in the [Scatterplots exercise notebook](2_Advanced/12_Exercise_Scatterplots.ipynb).

> [!CAUTION]
> **Salary-subset limitation:** Both prominence and median salary are calculated only from DACH postings containing `salary_year_avg`. The 8% threshold improves comparability within that subset but does not make it representative of all DACH job postings.

### Results

![Salary versus Prominence of Job Postings for Top Skills](images/Salary_vs_Prominence_of_Job_Postings_for_Top_Skills.png)

*Median annual salary versus the share of salary-reported DACH job postings mentioning each skill. Only skills reaching the 8% prominence threshold are displayed.*

### Insights

- **Spark and AWS offer the strongest visible balance:** Both are associated with median salaries of roughly **$147,500** while appearing in approximately **24%** and **20%** of the salary-reported postings respectively. They occupy the most attractive upper-right area of the chart.
- **Python and SQL provide the broadest applicability:** Python appears in more than **50%** of the salary subset and SQL in approximately **39%**. Their median salaries are lower than those of Spark and AWS, at roughly **$111,000**, but their much greater prominence makes them foundational skills.
- **Git combines a relatively high salary with lower prominence:** Its median is approximately **$131,000**, but it appears in only around **9%** of the salary-reported postings.
- **Azure and Tableau occupy the middle of the comparison:** Azure reaches about **12%** prominence with a median near **$111,000**, while Tableau is close to **11%** with a median around **$105,000**.
- **Kubernetes has the lowest observed median among the displayed skills:** It passes the prominence threshold but is associated with a median salary of approximately **$89,000** in this subset.
- **“Optimal” is a two-dimensional interpretation:** The notebook does not calculate a single composite score. Skills are evaluated visually by their position on the prominence and median-salary axes, so the preferred skill depends on whether breadth of demand or compensation receives more weight.
- **The findings remain directional:** Salary disclosure is limited, and role seniority, country, employer, and combinations with other skills can influence the observed medians. The chart supports prioritization hypotheses rather than causal conclusions about the salary effect of learning an individual skill.

## What I Learned So Far

- **Choosing the correct denominator matters:** Skill demand should be calculated against the number of original job postings, not against the number of rows produced by `explode()`.
- **Exploding list columns enables grouped analysis:** Converting each skill list into individual rows makes it possible to count and compare skill occurrences efficiently.
- **Percentages can overlap:** Since a single posting can request multiple skills, posting-level skill percentages do not need to sum to 100%.
- **Regional filtering changes the analytical context:** Findings for the DACH region should not be copied from a U.S.-focused analysis without recalculation.
- **Clear chart annotations improve readability:** Direct percentage labels make the comparison of skill demand easier across roles.
- **Monthly counts need normalization:** Dividing monthly skill mentions by the number of postings in the same month makes periods with different posting volumes comparable.
- **A fixed skill selection improves trend comparisons:** Selecting the five leading skills by their annual totals keeps the plotted categories consistent across all twelve months.
- **Boxplots and numerical summaries complement each other:** The table of counts and quartiles makes sample size limitations visible and explains why some median lines overlap the box boundaries.
- **Salary rankings need observation thresholds:** A high median based on only a few postings is less reliable than a similar median supported by many salary observations.
- **Missing salary values change the analytical population:** Salary-based role and skill rankings describe only the subset of postings that disclose annual compensation.
- **Raw dimensions require semantic checks:** The EDA location field mixes cities, countries, and remote labels, while Boolean columns such as `job_no_degree_mention` must be interpreted according to their exact definitions.
- **Entity resolution changes aggregated results:** Consolidating employer aliases prevents one organization from being split across several company labels.
- **An “optimal” skill depends on the objective:** Python and SQL maximize broad applicability, whereas Spark and AWS provide the strongest salary-prominence balance within the analyzed salary subset.

## Challenges

- Ensuring that the percentage denominator represents job postings rather than exploded skill rows.
- Handling postings with missing skill information without treating missing values as actual skills.
- Keeping the same visual scale across subplots while preserving readable labels.
- Separating supported conclusions from contextual fields that are not directly comparable or are poorly suited to the DACH region.
- Distinguishing changes in raw posting volume from changes in the percentage of postings that request a particular skill.
- Placing direct labels at the end of the trend lines without reducing readability or confusing closely positioned series.
- Interpreting salary distributions from relatively small samples without overstating differences between roles.
- Comparing specialized and frequently requested skills when their salary medians can be based on very different numbers of observations.
- Keeping the distinction clear between overall skill demand and skill frequency within the smaller salary-reported subset.
- Comparing city, country, and remote location labels that coexist in the same source column.
- Interpreting employment-related Boolean flags without reversing their meaning or overstating what an uninformative field can show.
- Balancing prominence and compensation without implying that one skill independently causes a higher salary.

## Conclusion

The exploratory analysis shows that Berlin and Vienna are the leading city-level location labels and that Deutsche Bahn is the most frequently represented employer after its naming variants are consolidated. It also demonstrates the importance of reading source fields carefully: the location ranking mixes geographical levels, and the health-insurance indicator is not suitable for drawing conclusions about actual benefit coverage in the DACH region.

The skill-demand analyses show that Python and SQL are central across the selected DACH data roles, while role-specific patterns remain visible: analyst roles place greater emphasis on reporting and visualization tools, engineering roles on cloud and distributed-processing technologies, and data-science roles on Python and R.

The monthly trend analysis confirms that Python and SQL remain the leading skills throughout 2023, even though their share of monthly postings declines during the second half of the year. Azure is comparatively stable before a fourth-quarter decrease, while R and AWS show more fluctuation and finish the year below their earlier levels.

The salary analysis shows substantial differences between the observed role medians and highlights AWS and Spark as comparatively well-paid among the frequently mentioned skills. At the same time, limited salary coverage and small group sizes reduce the reliability of individual rankings. The identical median salaries observed for Data Engineers and Senior Data Engineers illustrate why these findings should be treated as directional rather than definitive market benchmarks.

The final salary-versus-prominence analysis identifies Spark and AWS as the strongest visible balance of the two dimensions within the salary subset. Python and SQL remain the broadest foundational choices, while Git offers a comparatively high median with lower prominence. These outcomes are not universal skill rankings: they depend on the selected 8% threshold, the limited availability of salary information, and the relative importance assigned to prominence versus compensation.

Together, the five notebooks provide a complete exploratory workflow for the DACH data-job dataset—from market orientation and demand patterns to salary distributions and skill prioritization—while keeping the principal data-quality and sample-size limitations explicit.
