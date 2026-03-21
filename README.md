End-to-End Data Analysis Report: Nashville Housing Dataset
Project Overview

This project presents a complete end-to-end data analysis pipeline conducted on a real-world Nashville housing dataset containing 56,477 property transactions recorded between 2013 and 2016. The analysis focuses on transforming raw, inconsistent data into meaningful business insights through systematic data cleaning, feature engineering, exploratory analysis, and visualisation.

A key strength of this project is the implementation of the entire analytical pipeline in two separate environments—T-SQL (SQL Server) and Python (pandas, NumPy, matplotlib). This demonstrates the ability to apply consistent analytical thinking across different technologies, a highly valuable skill in real-world data roles.

Business Problem

The dataset in its raw form is not suitable for analysis due to multiple data quality issues such as missing values, duplicate records, inconsistent formats, and unstructured fields.

This project addresses critical business questions, including:

Which cities generate the highest total profit from property sales?
Which property types are most frequently sold and most profitable?
What are the highest-performing and worst-performing property investments?
How do property values vary across cities?
What characteristics define modern properties (post-2000 builds)?
Is there a relationship between property age and profitability?
What proportion of properties are sold as vacant versus occupied?
Dataset Description
Source: Nashville Housing Dataset (Kaggle)
Total Records: 56,477
Original Columns: 19
Final Columns: 21 (after feature engineering)
Date Range: April 2013 – September 2016
Key Variables
ParcelID: Unique property identifier
SalePrice: Transaction price
TotalValue: Assessed property value
LandValue / BuildingValue: Value breakdown
LandUse: Property type classification
SoldAsVacant: Vacancy status
YearBuilt: Construction year
Bedrooms, Bathrooms: Property characteristics
Data Quality Issues Identified

The dataset required extensive preprocessing due to:

Missing property addresses (29 records)
Large-scale missing owner and financial data (~31,000+ rows)
SaleDate stored as text instead of date format
Inconsistent categorical values (Y/N vs Yes/No)
Duplicate records (~103–104 rows)
Combined address fields requiring parsing
Data Cleaning and Transformation
SQL Server Approach

The SQL pipeline followed a structured sequence:

Converted SaleDate into a proper date format using CONVERT
Imputed missing PropertyAddress values via self-join on ParcelID
Split address fields using PARSENAME after string transformation
Standardised categorical values using CASE WHEN
Removed duplicates using ROW_NUMBER() window function
Dropped irrelevant columns to optimise the dataset

Created a derived column:

Profit_amount = SalePrice − TotalValue

Python Approach

The Python implementation replicated the same logic:

Converted dates using pd.to_datetime()
Handled missing addresses using sorting and forward fill
Split address fields using str.split()
Standardised categorical values using .replace()
Removed duplicates using .drop_duplicates()
Created the same profitability feature

This dual implementation highlights strong cross-platform analytical capability.

Exploratory Data Analysis (EDA)

The cleaned dataset was analysed using aggregation, grouping, ranking, and statistical methods to extract business insights.

Key Findings
1. Profit Distribution by City

Nashville dominates the dataset, contributing over 97% of total profit, significantly outperforming surrounding suburbs. This indicates a highly centralised real estate market.

2. Property Type Analysis
Single Family homes account for approximately 60.5% of all transactions
They also generate the highest total profit, making them the most commercially significant category
3. Best and Worst Investments
The highest profit recorded was $12.3M from vacant residential land
The worst-performing properties showed losses exceeding $5M, primarily due to sale prices being far below assessed value
4. Property Value Insights
Cities like Nolensville show extremely high average values, driven by a small number of high-value properties
This highlights the importance of considering distribution and sample size, not just averages
5. Vacancy Analysis
91.7% of properties were sold as occupied
Only 8.3% were vacant

This suggests strong occupancy demand in the market.

6. Modern Property Characteristics (Post-2000)

Newer properties tend to have:

More bedrooms and bathrooms
Larger overall profiles compared to early 2000s homes
7. Market Trend (2013–2016)

There was a sharp increase in property value between 2013 and 2015, reflecting a real estate boom, followed by stabilisation.

8. Profitability vs Property Age
Pearson correlation: r = -0.1532
Indicates a weak negative relationship

This suggests newer properties do not necessarily generate higher profits, largely due to higher initial valuations reducing margins.

Visualisations

The project includes multiple visualisations to support insights:

Bar charts for city-level comparisons
Pie chart for vacancy distribution
Histogram showing sale price distribution (right-skewed)
Line chart for temporal trends
Scatter plot for correlation analysis
Project Workflow

Raw Data → Data Cleaning (SQL & Python) → Feature Engineering → Exploratory Analysis → Visualisation → Business Insights

Tools and Technologies
SQL Server (T-SQL): Data cleaning and querying
Python: Parallel data pipeline and analysis
pandas: Data manipulation
NumPy: Statistical computation
matplotlib: Visualisation
Jupyter Notebook: Development environment
Skills Demonstrated

This project showcases:

Real-world data cleaning on large datasets
Handling missing values using relational and sequential logic
Deduplication using advanced SQL window functions
String parsing and feature extraction
Aggregation, grouping, and ranking techniques
Feature engineering for business metrics
Statistical analysis using correlation
Data visualisation and storytelling
Cross-tool competency (SQL + Python)
Conclusion

This project demonstrates the ability to transform messy, real-world data into actionable insights using both SQL and Python. It reflects not only technical proficiency but also strong analytical thinking and business understanding.

The dual implementation highlights adaptability across tools, making this project highly relevant for data analyst roles where flexibility and problem-solving are essential.
