# 📊 Data Analyst Portfolio: ShopEasy Marketing Optimization Project (2024)

## 🧠 Business Problem Overview

ShopEasy, an online retail company, has launched multiple digital marketing campaigns but is experiencing a steady decline in both customer engagement and conversion rates. Despite increasing ad spend, ROI has dropped, and the team is struggling to understand customer behavior and feedback.

> This project explores and analyzes:
> - Campaign effectiveness
> - Customer engagement trends
> - Customer review sentiment

🔗 **[View the full business case presentation here](.assets/docs/Marketing Analytics Business Case (Clean).pptx)**

---

## 🎯 Objectives
- Increase conversion rates by analyzing funnel drop-offs
- Enhance engagement by identifying effective content
- Improve customer feedback scores using sentiment analysis

  ## 📁 Project Folder Structure

This project is organized into the following folders:

| Folder | Description |
|--------|-------------|
| [`docs/`](./docs) | Contains the full project brief and stakeholder presentation (`.pptx`) |
| [`sql_queries/`](./sql_queries) | SQL scripts for data cleaning, transformation, and validation |
| [`datasets/`](./datasets) | Raw and processed datasets used throughout the project (`.xlsx`, `.csv`) |
| [`python_sentiment_analysis_and_calendar_dax_script/`](./python_sentiment_analysis_and_calendar_dax_script) | Python script for sentiment analysis and a reusable DAX calendar script |
| [`dashboard_file/`](./dashboard_file) | Final Power BI dashboard file (`.pbix`) for visual insights |

Each folder aligns with a specific project phase — from data wrangling to stakeholder-ready visualization.

## 🛠️ Tools & Technologies Used

| Tool / Technology | Purpose |
|-------------------|---------|
| **Microsoft Excel** | Initial data exploration and cleanup |
| **SQL Server** | Data cleaning, transformation, and validation using T-SQL |
| **Python (NLTK, Pandas)** | Sentiment analysis on customer reviews using VADER |
| **Power BI** | Interactive dashboard creation and KPI visualization |
| **GitHub** | Version control and hosting the project portfolio |

## 🎯 Project Objectives & Goals

The primary goal of this project is to help ShopEasy improve its marketing performance through a data-driven analysis of customer behavior and feedback.

### 🔍 Key Objectives:

- **Increase Conversion Rates**  
  Identify key drop-off points in the customer journey and optimize the funnel to improve conversion.

- **Enhance Customer Engagement**  
  Determine which types of marketing content drive the highest levels of interaction (likes, clicks, shares).

- **Improve Customer Feedback Scores**  
  Analyze review sentiment to uncover recurring pain points and satisfaction drivers for customers.

- **Present Business-Ready Insights**  
  Develop an executive dashboard and deliver a presentation highlighting actionable recommendations based on findings.

## 🧭 Project Workflow

This project follows a structured data analytics lifecycle:

1. 📥 **Data Collection & Exploration** – Understand the structure and quality of data.
2. 🧹 **Data Cleaning & Transformation** – Prepare the data using SQL and Excel.
3. 💬 **Sentiment Analysis** – Enrich customer reviews using Python + VADER.
4. 📊 **Dashboard Development** – Visualize KPIs and trends using Power BI.
5. 📈 **Insights & Recommendations** – Present business-ready conclusions.
6. 🗂️ **Documentation & Delivery** – Share results and files via GitHub.

---

## 📚 Table of Contents

- [Business Problem Overview](#-business-problem-overview)
- [Folder Structure](#project-folder-structure)
- [Tools & Technologies Used](#️-tools--technologies-used)
- [Objectives & Goals](#-project-objectives--goals)
- [Data Collection & Cleaning](#-data-collection--cleaning)
- [Sentiment Analysis (Python)](#-sentiment-analysis-python)
- [Dashboard Development (Power BI)](#-dashboard-development-power-bi)
- [Insights & Recommendations](#-insights--recommendations)
- [Final Presentation](#-final-presentation)

## 🧼 Data Collection & Cleaning

### 📥 Data Source

The dataset was restored from a SQL Server backup file:

📂 [`PortfolioProject_MarketingAnalytics.bak`](./datasets/PortfolioProject_MarketingAnalytics.bak)

This includes several normalized tables such as:
- `customers`
- `geography`
- `products`
- `customer_journey`
- `customer_reviews`
- `engagement_data`

---

### 1. 🧍‍♂️ Customer Table Enrichment

We joined the `customers` table with the `geography` table to enrich each record with the customer's `Country` and `City`.

```sql
SELECT 
    c.CustomerID,
    c.CustomerName,
    c.Email,
    c.Gender,
    c.Age,
    g.Country,
    g.City
FROM 
    dbo.customers AS c
LEFT JOIN 
    dbo.geography AS g
ON 
    c.GeographyID = g.GeographyID;
```


📸 *Result Screenshot:*  
![Customer Enrichment Result](./images/customer_enrichment.png)

---

### ✅ 2. Product Price Categorization

We used a `CASE` statement to categorize products into `Low`, `Medium`, and `High` based on their price.

```sql
SELECT 
    ProductID,
    ProductName,
    Price,
    CASE
        WHEN Price < 50 THEN 'Low'
        WHEN Price BETWEEN 50 AND 200 THEN 'Medium'
        ELSE 'High'
    END AS PriceCategory
FROM 
    dbo.products;
```  

📸 *Result Screenshot:*  
![Customer Enrichment Result](./images/customer_enrichment.png)



---

### ✅ 3. Remove Duplicates in Customer Journey

We used a CTE with `ROW_NUMBER()` to remove duplicate rows and filled missing `Duration` values using the average duration for each date.

```sql
WITH DuplicateRecords AS (
    SELECT 
        JourneyID, CustomerID, ProductID, VisitDate, Stage, Action, Duration,
        ROW_NUMBER() OVER (
            PARTITION BY CustomerID, ProductID, VisitDate, Stage, Action
            ORDER BY JourneyID
        ) AS row_num
    FROM dbo.customer_journey
)
SELECT 
    JourneyID, CustomerID, ProductID, VisitDate,
    UPPER(Stage) AS Stage,
    Action,
    COALESCE(Duration, avg_duration) AS Duration
FROM (
    SELECT *, 
        AVG(Duration) OVER (PARTITION BY VisitDate) AS avg_duration,
        ROW_NUMBER() OVER (
            PARTITION BY CustomerID, ProductID, VisitDate, UPPER(Stage), Action
            ORDER BY JourneyID
        ) AS row_num
    FROM dbo.customer_journey
) AS subquery
WHERE row_num = 1;
```  
📸 *Result Screenshot:*  
![Customer Enrichment Result](./images/customer_enrichment.png)


---

### ✅ 4. Review Text Cleanup

We standardized the review text by replacing double spaces with single spaces.

```sql
SELECT 
    ReviewID,
    CustomerID,
    ProductID,
    ReviewDate,
    Rating,
    REPLACE(ReviewText, '  ', ' ') AS ReviewText
FROM dbo.customer_reviews;

```

📸 *Result Screenshot:*  
![Customer Enrichment Result](./images/customer_enrichment.png)


---

### ✅ 5. Engagement Data Formatting

We cleaned and formatted the `engagement_data` table by:
- Fixing inconsistent `ContentType` values
- Splitting `ViewsClicksCombined` into separate `Views` and `Clicks`
- Formatting the `EngagementDate` into `dd.MM.yyyy`
- Filtering out irrelevant content types (e.g., Newsletters)

```sql
SELECT 
    EngagementID,
    ContentID,
    CampaignID,
    ProductID,
    UPPER(REPLACE(ContentType, 'Socialmedia', 'Social Media')) AS ContentType,
    LEFT(ViewsClicksCombined, CHARINDEX('-', ViewsClicksCombined) - 1) AS Views,
    RIGHT(ViewsClicksCombined, LEN(ViewsClicksCombined) - CHARINDEX('-', ViewsClicksCombined)) AS Clicks,
    Likes,
    FORMAT(CONVERT(DATE, EngagementDate), 'dd.MM.yyyy') AS EngagementDate
FROM dbo.engagement_data
WHERE ContentType != 'Newsletter';
```
📸 *Result Screenshot:*  
![Customer Enrichment Result](./images/customer_enrichment.png)

## 🧠 Sentiment Analysis with Python

We performed sentiment analysis on the customer reviews using **NLTK’s VADER Sentiment Analyzer** in Python. The goal was to understand customer sentiment beyond numeric ratings by analyzing review text.

---

### 🔍 What We Did:
- Computed compound sentiment scores for each review
- Categorized sentiment using both the score and customer rating
- Created sentiment buckets for better emotional grouping

📄 **Script Location**:  
[`customer_reviews_enrichment.py`](./python_sentiment_analysis_and_calendar_dax_script/customer_reviews_enrichment.py)

---

📸 *Result Screenshot:*  
![Sentiment Analysis Output](./images/sentiment_analysis_output.png)

## 📊 Dashboard Development (Power BI)

We created an interactive dashboard in **Power BI** to help stakeholders visualize and explore key performance indicators (KPIs) related to YouTube marketing performance.

The dashboard answers critical business questions, such as:
- Who are the top-performing YouTubers?
- What are their engagement rates?
- How do views, subscribers, and uploads relate to one another?

---

📄 **Dashboard File**:  
[`Dashboard.pbix`](./dashboard_file/Dashboard.pbix)

---

📸 *Dashboard Preview:*  
![Power BI Dashboard](./images/top_uk_youtubers_2024.gif)

## 💡 Insights & Recommendations

We concluded this analysis with a strategic focus on improving three core metrics: conversion rates, customer engagement, and customer feedback scores.

📄 **Full Insight Presentation**:  
[Marketing Insights & Recommendations (PPTX)](./docs/Marketing_Insights_ShopEasy.pptx)

---

### 🎯 1. Increase Conversion Rates

- **Goal**: Identify friction points in the funnel and provide actionable ways to improve conversions.
- **Insight**: Drop-off rates are highest at product pages and cart abandonment stages.
- **Action**: 
  - **Target High-Performing Products**: Focus on converting traffic for categories like *Kayaks*, *Ski Boots*, and *Baseball Gloves*.
  - **Use Seasonal Campaigns**: Promote during high-conversion months (e.g., January, September).
  - **Implement Personalized Campaigns**: Based on past user activity and reviews.

---

### 📈 2. Enhance Customer Engagement

- **Goal**: Determine what types of content generate the highest engagement across platforms.
- **Insight**: Engagement is low in Q4 (September–December); video content and blog CTAs underperform.
- **Action**:
  - **Revitalize Content Strategy**: Introduce **interactive videos**, polls, and **user-generated content**.
  - **Optimize CTAs**: Test CTA placement across blog posts and social media during low-engagement months.

---

### 💬 3. Improve Customer Feedback Scores

- **Goal**: Uncover review themes and improve overall satisfaction scores.
- **Insight**: Mixed reviews highlight recurring issues like delivery delays and unclear pricing.
- **Action**:
  - **Establish Feedback Loop**: Regularly analyze negative reviews and resolve underlying issues.
  - **Follow-Up with Dissatisfied Customers**: Encourage issue resolution + re-rating to raise the average toward the 4.0 benchmark.

---

📸 *Visual Snapshot from Insights Slide:*  
![Insight Visual](./images/insight_summary_slide.png)


