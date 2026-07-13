"# Capstone Project - Part 1" 
# Retail Sales Data Preparation & Exploratory Data Analysis (EDA)

Dataset : Kaggle
Notebook env: Jupyter Notebook/JupyterLab
Liabaries: pandas, numpy, os
visualization: seaborn, matplotlib

# 1. Data Cleaning and Memory Management Summary

Row Retention and Integrity: No rows were deleted or removed from the dataset during the cleaning process. The dataset maintains its original size of 500 rows. Because no rows were removed, the baseline baseline column null percentages were unaffected by data deletion. Instead, all initial missing values and subsequent type-conversion anomalies were handled strictly via imputation.

Memory Usage Comparison: Memory usage before and after datatype conversion was tracked using df.memory_usage(deep=True).sum().

Memory Usage Before: 85,233 bytes

Memory Usage After: 85,233 bytes

Net Savings: 0 bytes (0.00% reduction). Note: Converting product_name to a category optimized string retention, but because the size of the vocabulary relative to the small 500-row length of the series was balanced, the overall system memory footprint remained identical.

# 2. Skewness and Missing Value Strategy

Skewness of Columns:

quantity: 22.22 (Highly Positively Skewed)

sales_amount: 3.44 (Positively Skewed)

profit: 2.67 (Positively Skewed)

unit_price: 2.07 (Positively Skewed)

discount_pct: 0.05 (Approximately Symmetric)

# Positive vs. Negative Skew Definitions

Positive Skew (Right-Skewed): A distribution where the right tail is prolonged or stretched out. Extreme, high-value data points pull the mean upward to the right of the peak, resulting in a metric where $\text{Mean} > \text{Median}$.

Negative Skew (Left-Skewed): A distribution where the left tail is prolonged or stretched out. Extreme, low-value data points pull the mean downward to the left of the peak, resulting in a metric where $\text{Mean} < \text{Median}$.

# 3.Outlier Profile and Management Plan

An Interquartile Range (IQR) analysis with a traditional threshold of $\pm 1.5 \times \text{IQR}$ identified the following outlier characteristics:
    -> sales_amount Outliers: 68 instances representing 13.60% of total data points.
    -> profit Outliers: 48 instances representing 9.60% of total data points.


# 4. Visualization Insights & Descriptions

A. Distribution of the Most Skewed Column (quantity)
  
  -> Plot Reference: Generated via sns.histplot(df['quantity'], bins=20).

Shape Description: The histogram displays an extreme right-skewed "L-shape" or spike distribution. The vast majority of retail purchases reside tightly clustered in the single digits range (with a 25th percentile of 3, median of 5, and 75th percentile of 8 items). However, the distribution is stretched out by isolated wholesale-sized anomalies reaching a maximum volume of 999 units in a single order.

B. Correlation and Bivariate Relationship (sales_amount vs. profit)
    
   -> Plot Reference: Generated via sns.scatterplot(x='sales_amount', y='profit', data=df).

Direction and Strength: The scatter plot reveals a strong, positive, linear relationship. As sales_amount increases, profit scales upward systematically. The strength of the relationship is tightly clustered with minimal spread at low to mid-range values, maintaining a clear predictable direction even into high-leverage outlier sales amounts.

C. Categorical Stratification Box Plot (sales_amount by product_name)
 
    -> Plot Reference: Generated via sns.boxplot(x='product_name', y='sales_amount', data=df).

Median and Spread Differences: The box plot reveals massive structural differences across product types. Everyday commodities (e.g., Sugar, Sunscreen, Cooking Oil) display extremely narrow vertical spreads, with medians and IQRs pinned close to zero due to their low unit prices. Conversely, premium structural furniture items (e.g., Sofa, Bed Frame) exhibit expansive vertical spreads, vast interquartile ranges, and highly elevated medians, confirming that category membership is a primary driver of overall sales scale.

# 5. Categorical Group Feature Analytics
Grouping the target variable sales_amount by the categorical feature product_name provided the following statistical indicators:

  * Group with Highest Mean: Bed Frame (Mean: 22,071.92)

  * Group with Highest Standard Deviation (Std Dev): Sofa (driven by highly varied custom configurations and unit premiums)
