# Complete Guide: Building a Snowflake Dashboard for Food Truck Analytics

## Table of Contents
1. [Introduction](#introduction)
2. [Understanding Your Data](#understanding-your-data)
3. [Writing SQL Queries for Dashboard Insights](#writing-sql-queries-for-dashboard-insights)
4. [Building Your Dashboard](#building-your-dashboard)
5. [Advanced Dashboard Features](#advanced-dashboard-features)
6. [Best Practices](#best-practices)
7. [Troubleshooting](#troubleshooting)

---

## Introduction

Welcome to this comprehensive tutorial on building a Snowflake dashboard! In this guide, you'll learn how to create an interactive business intelligence dashboard using Snowflake's built-in visualization tools. We'll be working with a food truck business dataset that includes customer loyalty metrics, order details, and customer reviews.

**What you'll build:** A complete analytics dashboard showing:
- Sales performance across different countries and time periods
- Customer loyalty and behavior patterns
- Food truck performance and review analysis
- Revenue trends and forecasting insights

**Time to complete:** 1-2 hours

**Prerequisites:** 
- You're already logged into Snowflake
- The database `FOOD_TRUCK_ANALYTICS` with schema `DASHBOARD_DATA` exists
- Tables are created and populated with data
- Basic understanding of SQL (we'll explain concepts as we go)

### Business Context
Our dataset represents a food truck franchise with:
- **Multiple locations** across different cities and countries
- **Customer data** including loyalty metrics and purchase history
- **Order transactions** with detailed item-level information
- **Customer reviews** for quality and service feedback

---

## Understanding Your Data

Let's start by understanding the three main data sources we'll be working with:

### 1. Customer Loyalty Metrics (`CUSTOMER_LOYALTY_METRICS_V`)
This table contains customer profile and loyalty information:

| Column | Type | Description |
|--------|------|-------------|
| `CUSTOMER_ID` | NUMBER | Unique customer identifier |
| `CITY` | VARCHAR | Customer's city |
| `COUNTRY` | VARCHAR | Customer's country |
| `FIRST_NAME` | VARCHAR | Customer's first name |
| `LAST_NAME` | VARCHAR | Customer's last name |
| `PHONE_NUMBER` | VARCHAR | Contact phone number |
| `E_MAIL` | VARCHAR | Email address |
| `TOTAL_SALES` | NUMBER | Total lifetime sales to this customer |
| `VISITED_LOCATION_IDS_ARRAY` | ARRAY | List of locations the customer has visited |

**Key Insight:** This table helps us understand customer value and geographic distribution.

### 2. Orders (`ORDERS_V`)
This is our main transactional data containing detailed order information:

| Column | Type | Description |
|--------|------|-------------|
| `DATE` | DATE | Order date |
| `ORDER_ID` | NUMBER | Unique order identifier |
| `TRUCK_ID` | NUMBER | Which food truck served the order |
| `ORDER_TS` | TIMESTAMP | Exact time of order |
| `CUSTOMER_ID` | NUMBER | Links to customer loyalty table |
| `TRUCK_BRAND_NAME` | VARCHAR | Brand/franchise name |
| `MENU_TYPE` | VARCHAR | Type of cuisine |
| `PRIMARY_CITY` | VARCHAR | City where order was placed |
| `COUNTRY` | VARCHAR | Geographic country |
| `COUNTRY` | VARCHAR | Country of operation |
| `MENU_ITEM_NAME` | VARCHAR | Specific item ordered |
| `QUANTITY` | NUMBER | Number of items |
| `UNIT_PRICE` | NUMBER | Price per item |
| `ORDER_TOTAL` | NUMBER | Total order value |

**Key Insight:** This is our fact table - it contains the measurable business events we want to analyze.

### 3. Truck Reviews (`TRUCK_REVIEWS_V`)
Customer feedback and review data:

| Column | Type | Description |
|--------|------|-------------|
| `REVIEW_ID` | NUMBER | Unique review identifier |
| `ORDER_ID` | NUMBER | Links to specific order |
| `TRUCK_ID` | NUMBER | Which truck was reviewed |
| `CUSTOMER_ID` | NUMBER | Who left the review |
| `LANGUAGE` | VARCHAR | Language of the review |
| `SOURCE` | VARCHAR | Where the review came from |
| `REVIEW` | VARCHAR | Actual review text |
| `DATE` | DATE | When review was submitted |
| `TRUCK_BRAND_NAME` | VARCHAR | Brand that was reviewed |

**Key Insight:** This helps us measure customer satisfaction and identify improvement opportunities.

Before we start building our dashboard, let's make sure we're connected to the right database and schema:

```sql
-- Connect to the food truck analytics database
USE DATABASE FOOD_TRUCK_ANALYTICS;
USE SCHEMA DASHBOARD_DATA;

-- Verify our tables exist and have data
SELECT 'CUSTOMER_LOYALTY_METRICS_V' as TABLE_NAME, COUNT(*) as ROW_COUNT FROM CUSTOMER_LOYALTY_METRICS_V
UNION ALL
SELECT 'ORDERS_V' as TABLE_NAME, COUNT(*) as ROW_COUNT FROM ORDERS_V  
UNION ALL
SELECT 'TRUCK_REVIEWS_V' as TABLE_NAME, COUNT(*) as ROW_COUNT FROM TRUCK_REVIEWS_V;
```

**💡 Getting Started Tip:** Run the query above to confirm you can access all three tables and see how much data is available for analysis.

---

## Writing SQL Queries for Dashboard Insights

Before building visualizations, let's write SQL queries that will power our dashboard. Each query will answer a specific business question.

### Query 1: Daily Sales Trends

```sql
-- Daily sales performance over time
SELECT 
    DATE,
    COUNT(DISTINCT ORDER_ID) as TOTAL_ORDERS,
    SUM(ORDER_TOTAL) as DAILY_REVENUE,
    AVG(ORDER_TOTAL) as AVERAGE_ORDER_VALUE,
    COUNT(DISTINCT CUSTOMER_ID) as UNIQUE_CUSTOMERS
FROM ORDERS_V
GROUP BY DATE
ORDER BY DATE;
```

**Business Question:** How are our sales performing day by day?

### Query 2: Country Performance Analysis

```sql
-- Sales performance by country and city
SELECT 
    COUNTRY,
    PRIMARY_CITY,
    COUNT(DISTINCT ORDER_ID) as TOTAL_ORDERS,
    SUM(ORDER_TOTAL) as TOTAL_REVENUE,
    AVG(ORDER_TOTAL) as AVG_ORDER_VALUE,
    COUNT(DISTINCT CUSTOMER_ID) as UNIQUE_CUSTOMERS
FROM ORDERS_V
GROUP BY COUNTRY, PRIMARY_CITY
ORDER BY TOTAL_REVENUE DESC;
```

**Business Question:** Which countries and cities are generating the most revenue?

### Query 3: Menu Item Performance

```sql
-- Top performing menu items
SELECT 
    MENU_ITEM_NAME,
    MENU_TYPE,
    SUM(QUANTITY) as TOTAL_QUANTITY_SOLD,
    SUM(PRICE) as TOTAL_REVENUE,
    AVG(UNIT_PRICE) as AVERAGE_PRICE,
    COUNT(DISTINCT ORDER_ID) as ORDERS_CONTAINING_ITEM
FROM ORDERS_V
GROUP BY MENU_ITEM_NAME, MENU_TYPE
ORDER BY TOTAL_REVENUE DESC;
```

**Business Question:** What are our best-selling menu items?

### Query 4: Customer Loyalty Analysis

```sql
-- Customer loyalty and value analysis
SELECT 
    c.CUSTOMER_ID,
    c.FIRST_NAME,
    c.LAST_NAME,
    c.CITY,
    c.COUNTRY,
    c.TOTAL_SALES as LIFETIME_VALUE,
    COUNT(DISTINCT o.ORDER_ID) as TOTAL_ORDERS,
    AVG(o.ORDER_TOTAL) as AVERAGE_ORDER_SIZE,
    ARRAY_SIZE(c.VISITED_LOCATION_IDS_ARRAY) as LOCATIONS_VISITED
FROM CUSTOMER_LOYALTY_METRICS_V c
LEFT JOIN ORDERS_V o ON c.CUSTOMER_ID = o.CUSTOMER_ID
GROUP BY c.CUSTOMER_ID, c.FIRST_NAME, c.LAST_NAME, c.CITY, c.COUNTRY, 
         c.TOTAL_SALES, c.VISITED_LOCATION_IDS_ARRAY
ORDER BY c.TOTAL_SALES DESC;
```

**Business Question:** Who are our most valuable customers and what's their behavior?

### Query 5: Truck Brand Performance with Reviews

```sql
-- Truck performance including review sentiment
SELECT 
    o.TRUCK_BRAND_NAME,
    COUNT(DISTINCT o.ORDER_ID) as TOTAL_ORDERS,
    SUM(o.ORDER_TOTAL) as TOTAL_REVENUE,
    AVG(o.ORDER_TOTAL) as AVG_ORDER_VALUE,
    COUNT(DISTINCT r.REVIEW_ID) as TOTAL_REVIEWS,
    -- Simple sentiment analysis based on keywords
    SUM(CASE WHEN UPPER(r.REVIEW) LIKE '%GREAT%' OR UPPER(r.REVIEW) LIKE '%AMAZING%' 
             OR UPPER(r.REVIEW) LIKE '%EXCELLENT%' THEN 1 ELSE 0 END) as POSITIVE_REVIEWS,
    SUM(CASE WHEN UPPER(r.REVIEW) LIKE '%BAD%' OR UPPER(r.REVIEW) LIKE '%TERRIBLE%' 
             OR UPPER(r.REVIEW) LIKE '%AWFUL%' THEN 1 ELSE 0 END) as NEGATIVE_REVIEWS
FROM ORDERS_V o
LEFT JOIN TRUCK_REVIEWS_V r ON o.ORDER_ID = r.ORDER_ID
GROUP BY o.TRUCK_BRAND_NAME
ORDER BY TOTAL_REVENUE DESC;
```

**Business Question:** How do different truck brands perform in terms of sales and customer satisfaction?

**💡 SQL Learning Tip:** Notice how we use LEFT JOIN to combine data from multiple tables. This ensures we get all orders, even if they don't have reviews.

---

## Building Your Dashboard

Now comes the exciting part - creating visual dashboards in Snowflake! We'll follow the [official Snowflake dashboard documentation](https://docs.snowflake.com/en/user-guide/ui-snowsight-dashboards) to build our analytics dashboard.

### Step 1: Create Your Dashboard and First Tile

1. First, ensure you are using the correct role by clicking on your username in the top right corner and selecting **"Switch Role"**, then choose **"TB_ADMIN"**
2. In your Snowflake web interface, navigate to **Projects » Dashboards**
3. Click **"+ Dashboard"** to create a new dashboard
4. Give your dashboard a name: **"Food Truck Analytics Dashboard"**
5. Select a warehouse for the dashboard: choose **"TB_ANALYST_WH"** from the warehouse dropdown

Now let's create our first dashboard tile:

6. Click **"+"** (Add a dashboard tile)
7. Select **"New Tile from Worksheet"** - this opens a blank worksheet overlaying the dashboard
8. Select the database and schema from the dropdown: choose **"TB_101.ANALYTICS"**
9. In the SQL editor, paste our Daily Sales Trends query:

```sql
SELECT 
    DATE,
    COUNT(DISTINCT ORDER_ID) as TOTAL_ORDERS,
    SUM(ORDER_TOTAL) as DAILY_REVENUE,
    AVG(ORDER_TOTAL) as AVERAGE_ORDER_VALUE,
    COUNT(DISTINCT CUSTOMER_ID) as UNIQUE_CUSTOMERS
FROM ORDERS_V
GROUP BY DATE
ORDER BY DATE;
```

10. Run the SQL query to confirm it works and returns data
11. Click **"Chart"** to configure chart details
12. **Configure the Chart:**
    - Chart Type: **Line Chart**
    - X-Axis: **DATE**
    - Y-Axis: **DAILY_REVENUE**
    - Title: **"Daily Revenue Trend"**

13. Click **"Return to Food Truck Analytics Dashboard"** to save your worksheet and add it to the dashboard

**💡 Dashboard Tip:** Following Snowflake's documentation, tiles are flexible collections of charts that visualize your query results and can be customized and rearranged.

### Step 2: Explore Dashboard Filters and Create Custom Filter

Now let's explore the built-in filtering capabilities and add a custom filter:

1. In your dashboard, click **"Show or hide filter"** to reveal the filter panel
2. **Observe the system filters** that are automatically available:
   - **Date Range filter** (`:daterange`) - allows filtering by date periods
   - **Date Bucket filter** - allows grouping dates by day, week, month, etc.

3. **Create a custom Country filter:**
   - Click **"+ Filter"** to add a new custom filter
   - **Display Name:** `country`
   - **SQL Keyword:** `country`
   - **Warehouse:** Select **"DEFAULT_WH"** from the warehouse prompt
   - **Options via:** Select **"Query"** and click **"Write Query"**
   - In the **New filter window**, create this query:
   ```sql
   SELECT DISTINCT COUNTRY 
   FROM TB_101.RAW_POS.TRUCK 
   ORDER BY COUNTRY;
   ```
   - Click **"Done"**
   - Back on the add filter screen, change **Refresh** to **"Refresh daily"**
   - Select the options for **"Multiple values can be selected"** and **"Include an 'All' option"**
   - Click **"Save"**

**💡 Filter Tip:** Custom filters in Snowflake dashboards use parameter syntax (`:filtername`) and can be referenced in any query on the dashboard.

### Step 3: Edit Daily Sales Trend with Date Filter

Now let's modify our existing tile to use the date range filter:

1. From your Daily Revenue Trend tile, click the **tile menu (More options)** and select **"Edit query"**
2. **Update the query** to include date and country filters:

```sql
SELECT 
    DATE,
    COUNT(DISTINCT ORDER_ID) as TOTAL_ORDERS,
    SUM(ORDER_TOTAL) as DAILY_REVENUE,
    AVG(ORDER_TOTAL) as AVERAGE_ORDER_VALUE,
    COUNT(DISTINCT CUSTOMER_ID) as UNIQUE_CUSTOMERS
FROM ORDERS_V
WHERE DATE = :daterange AND COUNTRY = :country
GROUP BY DATE
ORDER BY DATE;
```

3. Run the query to test it works with the filter
4. **Notice no results were returned.** The default daterange filter is set for **"Last day"**. Click the filter in the top left (it will indicate "Last day") and change to **"Custom"**. Set a date range of **Jan 1, 2021** to **Dec 31, 2021**. Click **"Done"** and click **"Apply"**
5. Click **"Return to Food Truck Analytics Dashboard"** to save your changes

### Step 4: Add Country Performance Analysis Tile

1. Click **"+"** to add another dashboard tile
2. Select **"New Tile from Worksheet"**
3. Select **"TB_101.ANALYTICS"** database and schema
4. **Enter the Country Performance query with both filters:**

```sql
SELECT 
    COUNTRY,
    PRIMARY_CITY,
    COUNT(DISTINCT ORDER_ID) as TOTAL_ORDERS,
    SUM(ORDER_TOTAL) as TOTAL_REVENUE,
    AVG(ORDER_TOTAL) as AVG_ORDER_VALUE,
    COUNT(DISTINCT CUSTOMER_ID) as UNIQUE_CUSTOMERS
FROM ORDERS_V
WHERE DATE = :daterange 
  AND COUNTRY = :country
GROUP BY COUNTRY, PRIMARY_CITY
ORDER BY TOTAL_REVENUE DESC;
```

5. Run the query to confirm it works
6. Click **"Chart"** and configure:
   - Chart Type: **Bar Chart**
   - In the **Data** box, change **TOTAL_ORDERS** to **TOTAL_REVENUE**
   - Change the **Y-Axis** to **PRIMARY_CITY**
   - Click **+ Add column** to select **COUNTRY** column as a **Series** value
   - Set the **Title** of the chart to **"Revenue by City and Country"**

7. Click **"Return to Food Truck Analytics Dashboard"**

### Step 5: Create Total Revenue KPI Tile

1. Click **"+"** to add a new dashboard tile
2. Select **"New Tile from Worksheet"**
3. Select **"TB_101.ANALYTICS"** database and schema
4. **Enter the Total Revenue KPI query:**

```sql
SELECT SUM(ORDER_TOTAL) as TOTAL_REVENUE
FROM ORDERS_V
WHERE DATE = :daterange 
  AND COUNTRY = :country;
```

5. Run the query to test
6. Configure as a **single value display** (KPI format)
7. **Title:** "Total Revenue"
8. **Format:** Currency
9. Click **"Return to Food Truck Analytics Dashboard"**

### Step 6: Duplicate and Modify KPI Tiles

1. **Duplicate the Total Revenue tile:**
   - From the Total Revenue tile menu (More options), select **"Duplicate Tile"**
   - A copy appears at the bottom of the dashboard

2. **Create Total Orders KPI:**
   - From the duplicated tile menu, select **"Edit query"**
   - **Update the query:**
   ```sql
   SELECT COUNT(DISTINCT ORDER_ID) as TOTAL_ORDERS
   FROM ORDERS_V
   WHERE DATE = :daterange 
     AND COUNTRY = :country;
   ```
   - Update title to **"Total Orders"**
   - Click **"Return to Food Truck Analytics Dashboard"**

3. **Duplicate again for Average Order Value:**
   - Duplicate the Total Revenue tile again
   - Edit the query:
   ```sql
   SELECT ROUND(AVG(ORDER_TOTAL), 2) as AVERAGE_ORDER_VALUE
   FROM ORDERS_V
   WHERE DATE = :daterange 
     AND COUNTRY = :country;
   ```
   - Update title to **"Average Order Value"**
   - Format as Currency
   - Click **"Return to Food Truck Analytics Dashboard"**

### Step 7: Reposition KPI Tiles at Top of Dashboard

1. **Drag the KPI tiles to the top:**
   - Click and drag the **Total Revenue** tile to the top-left position
   - Drag the **Total Orders** tile next to it (top-center)
   - Drag the **Average Order Value** tile to the top-right
   - As you drag each tile, you'll see a preview of the new position

2. **Arrange the remaining tiles:**
   - Position the **Daily Revenue Trend** chart below the KPIs
   - Position the **Country Performance** chart at the bottom

**💡 Layout Tip:** Following Snowflake's documentation, you can rearrange tiles by dragging them to new positions, and a preview shows where the tile will be placed.

---

## Advanced Dashboard Features

### Adding Filters and Interactivity

Filters make your dashboard interactive and allow users to drill down into specific data.

#### Step 1: Add Date Range Filter

1. Click **"+ Filter"** in your dashboard
2. **Configure:**
   - Filter Type: **Date Range**
   - Column: **DATE** (from ORDERS_V table)
   - Default Range: **Last 30 days**
   - Title: **"Order Date Range"**

#### Step 2: Add Country Filter

1. Add another filter
2. **Configure:**
   - Filter Type: **List**
   - Column: **COUNTRY** (from ORDERS_V table)
   - Allow Multiple Selection: **Yes**
   - Title: **"Select Countries"**

#### Step 3: Add Menu Type Filter

1. Add a third filter
2. **Configure:**
   - Filter Type: **List**
   - Column: **MENU_TYPE** (from ORDERS_V table)
   - Title: **"Cuisine Type"**

**💡 Interactivity Tip:** When users select different filter values, all charts on the dashboard will automatically update to show only the filtered data.

### Creating Calculated Fields

Sometimes you need to create new metrics that don't exist in your raw data.

#### Revenue Growth Rate

```sql
SELECT 
    DATE,
    SUM(ORDER_TOTAL) as DAILY_REVENUE,
    LAG(SUM(ORDER_TOTAL)) OVER (ORDER BY DATE) as PREVIOUS_DAY_REVENUE,
    ((SUM(ORDER_TOTAL) - LAG(SUM(ORDER_TOTAL)) OVER (ORDER BY DATE)) / 
     LAG(SUM(ORDER_TOTAL)) OVER (ORDER BY DATE)) * 100 as GROWTH_RATE_PERCENT
FROM ORDERS_V
GROUP BY DATE
ORDER BY DATE;
```

#### Customer Segmentation

```sql
SELECT 
    CUSTOMER_ID,
    TOTAL_SALES,
    CASE 
        WHEN TOTAL_SALES >= 2000 THEN 'VIP Customer'
        WHEN TOTAL_SALES >= 1000 THEN 'High Value Customer'
        WHEN TOTAL_SALES >= 500 THEN 'Regular Customer'
        ELSE 'New Customer'
    END as CUSTOMER_SEGMENT
FROM CUSTOMER_LOYALTY_METRICS_V;
```

### Setting Up Automated Refresh

To keep your dashboard current with fresh data:

1. In your dashboard settings, find **"Auto Refresh"**
2. Set refresh interval (e.g., every hour, daily)
3. Choose the warehouse to use for refreshes
4. Save your settings

**💡 Cost Management:** Be mindful of refresh frequency - more frequent refreshes use more compute credits.

---

## Best Practices

### 1. Dashboard Design Principles

**Keep It Simple:**
- Limit to 5-7 key visualizations per dashboard
- Use consistent colors and fonts
- Group related charts together

**Tell a Story:**
- Arrange charts in logical order (overview → details)
- Use titles that explain what users should learn
- Include context with subtitles or descriptions

**Make It Actionable:**
- Include filters for interactivity
- Highlight important trends or outliers
- Provide drill-down capabilities

### 2. SQL Query Optimization

**Use Appropriate Aggregations:**
```sql
-- Good: Specific aggregation
SELECT DATE, SUM(ORDER_TOTAL) FROM ORDERS_V GROUP BY DATE;

-- Avoid: SELECT * without purpose
SELECT * FROM ORDERS_V; -- This loads all data unnecessarily
```

**Leverage Indexes and Clustering:**
```sql
-- Create clustering key for better performance on date queries
ALTER TABLE ORDERS_V CLUSTER BY (DATE);
```

**Use LIMIT for Testing:**
```sql
-- When developing queries, use LIMIT to avoid long wait times
SELECT * FROM ORDERS_V LIMIT 100;
```

### 3. Performance Optimization

**Choose Right Warehouse Size:**
- X-Small: Development and small datasets
- Small: Regular reporting
- Medium+: Large datasets or complex queries

**💡 Warehouse Tip:** Since your environment is already set up, you can check your current warehouse with `SELECT CURRENT_WAREHOUSE();` and resize if needed with `ALTER WAREHOUSE [name] SET WAREHOUSE_SIZE = 'SMALL';`

**Use Result Caching:**
- Snowflake automatically caches query results
- Identical queries return instantly from cache
- Cache lasts 24 hours

**Optimize Data Types:**
```sql
-- Use appropriate precision for numbers
NUMBER(10,2) -- Better than NUMBER(38,4) for prices
DATE -- Better than VARCHAR for dates
```

### 4. Security and Access Control

**Create Role-Based Access:**
```sql
-- Create roles for different user types
CREATE ROLE DASHBOARD_VIEWER;
CREATE ROLE DASHBOARD_ADMIN;

-- Grant appropriate permissions
GRANT SELECT ON ALL TABLES IN SCHEMA DASHBOARD_DATA TO ROLE DASHBOARD_VIEWER;
GRANT ALL ON SCHEMA DASHBOARD_DATA TO ROLE DASHBOARD_ADMIN;
```

**Share Dashboards Securely:**
- Use Snowflake's sharing features
- Set appropriate access levels
- Regular review of user permissions

---

## Troubleshooting

### Common Issues and Solutions

#### Issue 1: "Object does not exist" Error

**Problem:** You get an error saying a table or column doesn't exist.

**Solution:**
```sql
-- Check if you're in the right database and schema
SELECT CURRENT_DATABASE(), CURRENT_SCHEMA();

-- List all tables in current schema
SHOW TABLES;

-- Describe table structure
DESCRIBE TABLE ORDERS_V;
```

#### Issue 2: Charts Not Loading or Showing No Data

**Problem:** Your dashboard charts appear empty or won't load.

**Solutions:**
1. **Check your query results first:**
```sql
-- Test your query in a worksheet before adding to dashboard
SELECT COUNT(*) FROM ORDERS_V; -- Should return a number > 0
```

2. **Verify date formats:**
```sql
-- Check date column format
SELECT DATE, typeof(DATE) FROM ORDERS_V LIMIT 5;
```

3. **Check for NULL values:**
```sql
-- Count NULL values in key columns
SELECT 
    COUNT(*) as TOTAL_ROWS,
    COUNT(ORDER_TOTAL) as NON_NULL_ORDER_TOTAL,
    COUNT(DATE) as NON_NULL_DATES
FROM ORDERS_V;
```

#### Issue 3: Slow Dashboard Performance

**Problem:** Dashboard takes too long to load.

**Solutions:**
1. **Optimize your queries:**
```sql
-- Add WHERE clauses to limit data
SELECT * FROM ORDERS_V 
WHERE DATE >= CURRENT_DATE - 30; -- Only last 30 days
```

2. **Use appropriate warehouse size:**
```sql
-- Scale up warehouse temporarily
ALTER WAREHOUSE ANALYTICS_WH SET WAREHOUSE_SIZE = 'SMALL';
```

3. **Create summary tables:**
```sql
-- Create pre-aggregated tables for common queries
CREATE TABLE DAILY_SALES_SUMMARY AS
SELECT 
    DATE,
    SUM(ORDER_TOTAL) as DAILY_REVENUE,
    COUNT(DISTINCT ORDER_ID) as DAILY_ORDERS
FROM ORDERS_V
GROUP BY DATE;
```

#### Issue 4: Permission Denied Errors

**Problem:** You can't access certain features or data.

**Solution:**
```sql
-- Check your current role and privileges
SELECT CURRENT_ROLE();
SHOW GRANTS TO ROLE CURRENT_ROLE();

-- Switch to a role with more privileges if available
USE ROLE ACCOUNTADMIN; -- Use carefully, only if you have this role
```

#### Issue 5: Data Not Refreshing

**Problem:** Your dashboard shows old data even after new data is loaded.

**Solutions:**
1. **Clear result cache:**
```sql
-- Add a comment to force query re-execution
SELECT * FROM ORDERS_V -- Updated at 2024-01-16 10:30
```

2. **Check auto-refresh settings:**
   - Go to dashboard settings
   - Verify refresh schedule is active
   - Check if warehouse is suspended

3. **Manual refresh:**
   - Click the refresh button on individual charts
   - Or refresh the entire dashboard

### Getting Help

**Snowflake Documentation:**
- Official docs: docs.snowflake.com
- Community forums: community.snowflake.com

**SQL Learning Resources:**
- W3Schools SQL Tutorial
- SQLBolt interactive lessons
- Snowflake University (free courses)

**Dashboard Design Resources:**
- "Information Dashboard Design" by Stephen Few
- "Storytelling with Data" by Cole Nussbaumer Knaflic

---

## Conclusion

Congratulations! You've just built a comprehensive business intelligence dashboard in Snowflake. Here's what you've accomplished:

✅ **Set up a complete Snowflake environment** with databases, schemas, and warehouses
✅ **Created and populated data tables** based on real business requirements  
✅ **Written complex SQL queries** to extract business insights
✅ **Built interactive visualizations** showing sales trends, country performance, and customer behavior
✅ **Added filters and interactivity** to make your dashboard user-friendly
✅ **Learned best practices** for performance, security, and design
✅ **Gained troubleshooting skills** to solve common issues

### Next Steps

Now that you have a solid foundation, consider these advanced topics:

1. **Advanced Analytics:**
   - Time series forecasting
   - Customer churn prediction
   - Market basket analysis

2. **Integration:**
   - Connect external data sources
   - Set up automated data pipelines
   - Integrate with BI tools like Tableau or Power BI

3. **Advanced Snowflake Features:**
   - Snowpipe for real-time data loading
   - Tasks and streams for data processing
   - Data sharing with external organizations

4. **Machine Learning:**
   - Snowflake's ML functions
   - Integration with Python and R
   - Predictive analytics

### Remember

Building great dashboards is an iterative process. Start simple, gather feedback from users, and continuously improve. The most important thing is that your dashboard helps people make better business decisions.

Happy analyzing! 🚀

---

*This tutorial was created to help you get started with Snowflake dashboards. For the most up-to-date information, always refer to the official Snowflake documentation.*
