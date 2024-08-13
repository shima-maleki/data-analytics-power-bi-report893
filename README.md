# Power BI Project: Sales Data Analysis

This Power BI project aims to analyze sales data by integrating various data sources, performing data transformation, and leveraging Power BI's robust capabilities for data modeling and visualization. The end goal is to create an interactive and insightful report that showcases key metrics and trends in the company's sales performance.

![Alt text](images\arc.jpg)

## Table of Contents
1. Set up the Environment
2. Import the Data into Power BI
3. Create the Data Model
4. Set up the Report
5. Build the Customer Detail Page
6. Create an Executive Summary Page
7. Create a Product Detail Page
8. Create a Stores Map Page
9. Cross-Filtering and Navigation
10. Create Metrics for Users Outside the Company Using SQL

## 1. Set up the Environment
- Download Power BI desktop on Windows Machine
- Set Up a GitHub Repo for the project

## 2. Import the Data into Power BI

### 2.1. Import and prepare the orders_powerbi table from an Azure SQL Database, ensuring data integrity and privacy.

**Connect to Azure SQL Database:**

**Server Name:** 'Server-Name'

**Database Name:** powerbi-orders-db

**Username:** 'User-Name'

**Password:** 'Password'

**Data Preparation:**
- Remove Sensitive Data:
Deleted the [Card Number] column to ensure data privacy.
- Date and Time Separation:
Split the [Order Date] and [Shipping Date] columns into two distinct columns each.
- Data Integrity:
Filtered out and removed any rows where the [Order Date] column had missing or null values.
- Column Renaming:
Renamed columns to follow Power BI naming conventions, enhancing clarity and consistency in the report.

### 2.2. Use Power BI's Get Data option to import the Products.csv file into the project.

**Data Preparation:**
- Remove Duplicates:
Applied the Remove Duplicates function on the product_code column to ensure uniqueness.
- Column Renaming:
Renamed columns to align with Power BI naming conventions, ensuring consistency in presentation.

### 2.3. Import the Stores table from Azure Blob Storage and prepare it for analysis.

**Connect to Azure Blob Storage:**

- **Account Name:** 'AccountName'

- **Account Key:** 'AccountKey'

- **Container Name:** data-analytics

Used Power BI's Get Data option to connect to Azure Blob Storage and import the Stores table.

**Data Preparation:**
- Column Renaming:
Renamed columns to follow Power BI naming conventions for clarity and consistency.

### 2.4. Import and combine multiple CSV files from a folder into a single Customers table, ensuring consistency and data integrity.

Downloaded the Customers.zip file, unzipped it. Used the Folder data connector in Power BI to import the Customers folder. 

Combined and transformed the data from the three CSV files into one query, ensuring the data was correctly appended.

**Data Preparation:**

- Create Full Name Column:
Created a Full Name column by concatenating the [First Name] and [Last Name] columns using the following DAX formula:

```
Full Name = [First Name] & " " & [Last Name]
```

- Remove Unused Columns:
Deleted any obviously unused columns (e.g., index columns) to streamline the dataset.

- Column Renaming:
Renamed remaining columns to align with Power BI naming conventions for consistency.

## 3. Create the Data Model
To make use of Power BI's time intelligence functions, a continuous Date table was created, covering the entire period of the sales data.

- Identify the Date Range & Create the Date Table:

Generated a continuous Date table using the following DAX formula:
```
Date = 
CALENDAR(
    DATE(YEAR(MIN(Orders[Order Date])), 1, 1), 
    DATE(YEAR(MAX(Orders[Shipping Date])), 12, 31)
)
```

To enhance the functionality of the Date table, I added several calculated columns:
DAX
```
'Date'[Day of Week] = WEEKDAY('Date'[Date])
'Date'[Month Number] = MONTH('Date'[Date])
'Date'[Month Name] = FORMAT('Date'[Date], "MMMM")
'Date'[Quarter] = QUARTER('Date'[Date])
'Date'[Year] = YEAR('Date'[Date])
'Date'[Start of Year] = DATE('Date'[Year], 1, 1)
'Date'[Start of Quarter] = STARTOFQUARTER('Date'[Date])
'Date'[Start of Month] = STARTOFMONTH('Date'[Date])
'Date'[Start of Week] = 'Date'[Date] - WEEKDAY('Date'[Date], 1) + 1
```

The Date table now includes essential time-related columns, enabling robust time intelligence analysis in the subsequent tasks.

- Establishing Relationships and Building a Star Schema

I created the following relationships to form a star schema:

| From Table | From Column  | To Table | To Column     | Relationship Type |
|------------|--------------|----------|---------------|-------------------|
| Products   | product_code | Orders   | product_code  | One-to-Many       |
| Stores     | store code   | Orders   | Store Code    | One-to-Many       |
| Customers  | User UUID    | Orders   | User ID       | One-to-Many       |
| Date       | Date         | Orders   | Order Date    | One-to-Many       |
| Date       | Date         | Orders   | Shipping Date | One-to-Many       |

![Alt text](images\relationship.JPG)

- Creating a Separate Measures Table
To maintain an organized and efficient data model, a separate Measures table was created for storing all DAX measures. The Measures Table serves as a centralized location for all measures, improving the organization and manageability of the data model.

A set of key measures was created to provide foundational metrics for the sales analysis.
```
Total Orders = COUNTROWS(Orders)
Total Revenue = SUMX(Orders, Orders[Product Quantity] * RELATED(Products[Sale_Price]))
Total Profit = SUMX(Orders, (RELATED(Products[Sale_Price]) - RELATED(Products[Cost_Price])) * Orders[Product Quantity])
Total Customers = DISTINCTCOUNT(Orders[User ID])
Total Quantity = SUM(Orders[Product Quantity])
Profit YTD = TOTALYTD([Total Profit], 'Date'[Date])
Revenue YTD = TOTALYTD([Total Revenue], 'Date'[Date])
```

These measures provide a strong foundation for the sales analysis, enabling insights into orders, revenue, profit, and customer behavior over time.

- Creating Hierarchies
Hierarchies were created to allow for detailed drill-down capabilities in the reports, facilitating granular analysis.

The Date hierarchy was created with the following levels:

- **Start of Year**
- **Start of Quarter**
- **Start of Month**
- **Start of Week**
- **Date**

Geography Hierarchy:
A calculated column Country was added to the Stores table:
```
Country = SWITCH([Country Code], "GB", "United Kingdom", "US", "United States", "DE", "Germany")

Geography = [Country Region] & ", " & [Country]
```
The Geography hierarchy was created with the following levels:

- **World Region**

- **Country**

- **Country Region**


The correct data categories were assigned to the following columns:

- **World Region**: `Continent`

- **Country**: `Country`

- **Country Region**: `State or Province`

The hierarchies enable users to drill down into data by time and geography, enhancing the interactivity and depth of the analysis.

## 4. Set up the Report

Create the foundational structure of the Power BI report by setting up four main report pages and applying a consistent color theme across the entire report.

   - **Create Report Pages**:

     - **Executive Summary**: This page will provide a high-level overview of key business metrics and trends.#

     - **Customer Detail**: This page will focus on detailed customer data, including customer segmentation and behavior analysis.

     - **Product Detail**: This page will provide in-depth insights into product performance, including sales, profitability, and category trends.

     - **Stores Map**: This page will visually represent store performance across different geographic regions using map visuals.

   - **Select and Apply a Color Theme**:

     - Explored the available pre-defined themes in Power BI to find one that aligns with the branding and visual identity of the report.

   - **Add Navigation Sidebar**:

     - Added a rectangle shape on the Executive Summary page, covering a narrow strip on the left side of the page.

     - Set the fill color of the rectangle to a contrasting color that complements the chosen theme.

     - This sidebar will be used as a navigation panel to switch between different pages in the report.

   - **Duplicate Sidebar Across Pages**:

     - Duplicated the rectangle shape on each of the other report pages (Customer Detail, Product Detail, and Stores Map).

     - Ensured the sidebar maintains the same size, position, and color across all pages for a uniform look and feel.

## 5. Build the Customer Detail Page

![Alt text](images\customer.JPG)

Adding card visuals, charts, and tables that provide detailed and actionable insights into customer behavior and performance.
Detail Page

Create and configure a set of gauges, slicers, and advanced visuals to enhance the dashboard, providing clear and actionable insights into the company's quarterly performance, product profitability, and customer segmentation.

   - **Create and Configure Gauges for**

     - Added a card visual for the `[Total Customers]` measure.

   - **Revenue per Customer**:

     - Created a new measure in the `Measures Table` called `[Revenue per Customer]` using the following DAX formula:
       
       ```DAX
       Revenue per Customer = [Total Revenue] / [Total Customers]
       ```

     - Added a card visual for the `[Revenue per Customer]` measure, providing insights into average revenue per customer.

     - Added a Donut Chart visual to show the distribution of `Total Customers` across different countries.

     - Used the `Users[Country]` column to filter the `[Total Customers]` measure, providing a breakdown of customer base by geography.

   - **Add Column Chart for Customers by Product Category**:

     - Added a Column Chart visual to display the number of customers who purchased products from each category.

     - Used the `Products[Category]` column to filter the `[Total Customers]` measure, showing customer distribution across product categories.

   - **Add Line Chart for Customer Trends**:

     - Added a Line Chart visual at the top of the page.

     - **X-Axis**: Set to use the `Date Hierarchy`, allowing users to drill down from year to month.

     - **Y-Axis**: Set to `[Total Customers]`.

     - **Additional Features**:

       - Added a trend line to observe customer growth over time.

       - Added a forecast for the next 10 periods with a 95% confidence interval, helping to predict future customer trends.

   - **Create a Table for Top 20 Customers by Revenue**:

     - Created a new table to display the top 20 customers, filtered by revenue.

     - The table includes each customer’s full name, total revenue, and the number of orders.

   - **Add Card Visuals for Top Customer Insights**:

     - Created a set of three card visuals to provide insights into the top customer by revenue:

       - **Card 1**: Displays the top customer's name.

       - **Card 2**: Shows the number of orders made by the top customer.

       - **Card 3**: Displays the total revenue generated by the top customer.

   - **Add a Date Slicer**:

     - Added a date slicer to allow users to filter the page by year.

     - Set the slicer style to **Between**, enabling users to select a range of dates for the analysis.

## 6. Create an Executive Summary Page

![Alt text](images\executive-summary.JPG)

Create a comprehensive Executive Summary page in Power BI, consisting of key metrics and visualizations that provide a high-level overview of the business performance.


   - **Group Card Visuals for Key Metrics**

     - Assigned the cards to the following measures:

       - **Card 1**: `Total Revenue`

       - **Card 2**: `Total Orders`

       - **Card 3**: `Total Profit`


   - **Line Graph for Revenue Trends**:

     - Set the X-axis to the `Date Hierarchy`, displaying only the `Start of Year`, `Start of Quarter`, and `Start of Month` levels.

     - Set the Y-axis to `Total Revenue`


   - **Donut Charts for Revenue Breakdown**:

     - Added two donut charts to visualize `Total Revenue`

       - **Chart 1**: Broken down by `Store[Country]`

       - **Chart 2**: Broken down by `Store[Store Type]`

   - **Bar Chart for Orders by Product Category**

     - Copied the `Total Customers by Product Category` donut chart from the Customer Detail page.

     - Changed the visual type to `Clustered Bar Chart`

     - Changed the X-axis field from `Total Customers` to `Total Orders`

   - **Create KPIs for Quarterly Metrics**:

     - Created the following measures to calculate the previous quarter's metrics:

       - `Previous Quarter Profit`

       - `Previous Quarter Revenue`

       - `Previous Quarter Orders`

     - Created target measures with a 5% growth compared to the previous quarter:

       - `Target Revenue`

       - `Target Profit`

       - `Target Orders`

   - **Add Revenue KPI**:

     - Created a KPI visual for `Total Revenue`:

       - **Value Field**: `Total Revenue`

       - **Trend Axis**: `Start of Quarter`

       - **Target Field**: `Target Revenue`

   - **Duplicate and Configure for Profit and Orders**:

     - Duplicated the KPI visual two more times.
     
     - Configured the duplicates to display `Total Profit` and `Total Orders` with their respective targets and trends.

## 7. Create a Product  Quarterly Performance**:

![Alt text](images\product.JPG)

   - **DAX Measures for Quarterly Targets**:

     - Created DAX measures for the current quarter’s `Orders`, `Revenue`, and `Profit`.

     - Created corresponding target measures based on a 10% quarter-on-quarter growth, as requested by the CEO.

     - Defined measures for the gap between current performance and targets:

       ```DAX
       Gap Orders = [Current Quarter Orders] - [Target Orders]
       Gap Revenue = [Current Quarter Revenue] - [Target Revenue]
       Gap Profit = [Current Quarter Profit] - [Target Profit]
       ```

   - **Gauge Visuals**:

       - **Gauge 1**: `Orders`

       - **Gauge 2**: `Revenue`

       - **Gauge 3**: `Profit`
    
   - **Add Placeholder Shapes for Slicer State Cards**:

   - **Card Visuals for Slicer States**:

     - Defined DAX measures to display the current filter states:

       ```DAX
       Category Selection = IF(ISFILTERED(Products[Category]), SELECTEDVALUE(Products[Category], "No Selection"), "No Selection")
       Country Selection = IF(ISFILTERED(Stores[Country]), SELECTEDVALUE(Stores[Country],"No Selection"))
       ```

     - Added a card visual to rectangle and assigned the respective measures.

   - **Create an Area Chart for Product Category Revenue**:

     - Added a new area chart to display revenue performance over time by product category.

     - **X-Axis**: Set to `Dates[Start of Quarter]`.

     - **Y-Axis**: Set to `Total Revenue`.

     - **Legend**: Set to `Products[Category]`.

   - **Add a Top 10 Products Table**:

     - Added a table visual to display the top 10 products by revenue, with the following fields:

       - `Product Description`

       - `Total Revenue`

       - `Total Customers`

       - `Total Orders`

       - `Profit per Order`'

   - **Create a Scatter Chart for Product Profitability**:

     - Created a new calculated column `[Profit per Item]` in the Products table using the following DAX formula:

       ```DAX
       Profit per Item = [Sale Price] - [Cost Price]
       ```

   - **Scatter Chart**:

       - **Values**: `Products[Description]`

       - **X-Axis**: `Products[Profit per Item]`

       - **Y-Axis**: `Orders[Total Quantity]`

       - **Legend**: `Products[Category]`

## 8. Create a Stores Map Page

![Alt text](images\stores.JPG)

Enhance the Stores Map page with interactive visuals and create a drillthrough page for detailed store performance analysis, as well as a custom tooltip for quick insights.

   - **Add and Configure Map Visual on Stores Map Page**:

     - Added a new map visual that takes up the majority of the page, leaving a narrow band at the top for a slicer.

     - Assigned the `Geography` hierarchy to the `Location` field.

     - Assigned `Profit YTD` to the `Bubble size` field.

   - **Add a Slicer Above the Map**:

     - Added a slicer above the map, setting the slicer field to `Stores[Country]`.

   - **Create Drillthrough Page for Store Performance**:

   - **Page Setup**:

     - Created a new page named **Stores Drillthrough**.

     - Opened the Format pane and expanded the `Page information` tab.

     - Set the `Page type` to **Drillthrough**.

     - Set `Drill through when` to **Used as category** and `Drill through from` to **Country Region**.

   - **DAX Measures for Gauges**:

     - Used measures for `Profit YTD` and `Revenue YTD`.

     - Created new measures for `Profit Goal` and `Revenue Goal` to reflect a 20% increase on the previous year's year-to-date profit or revenue at the current point in the year:

       ```DAX
       Profit Goal = [Profit YTD] * 1.2
       Revenue Goal = [Revenue YTD] * 1.2
       ```

   - **Visuals Added**:

     - **Table**: Displaying the top 5 products with columns: `Description`, `Profit YTD`, `Total Orders`, `Total Revenue`.

     - **Column Chart**: Showing `Total Orders` by product category for the store.

     - **Gauges**: Displaying `Profit YTD` against the profit goal with the target set to a 20% year-on-year growth.

     - **Card Visual**: Showing the currently selected store.

   - **Create a Custom Tooltip for Store Profit Performance**:

   - **Custom Tooltip Page**:

     - Created a new tooltip page specifically for displaying store profit performance.

     - Copied the `Profit YTD` gauge visual to this tooltip page.

   - **Tooltip Configuration**:

     - Set the tooltip of the map visual on the Stores Map page to this newly created tooltip page.

     - This allows users to see each store's year-to-date profit performance against the profit target by simply hovering the mouse over a store on the map.

![Alt text](images\drillsthrough.JPG)

## 9. Cross-Filtering and Navigation

Fine-tune the interactions between visuals across different report pages to ensure that the filters and cross-filtering behave as expected, providing users with a consistent and logical experience.

   - **Executive Summary Page**:

   - **Product Category Bar Chart and Top 10 Products Table**:

     - **Interaction Setting**: These visuals were configured so that they do not filter the card visuals or KPIs on the Executive Summary page.

   - **Customer Detail Page**:

   - **Top 20 Customers Table**:

     - **Interaction Setting**: Configured so that the Top 20 Customers table does not filter any of the other visuals on the page.

   - **Total Customers by Product Donut Chart**:

     - **Interaction Setting**: Configured so that this chart does not affect the Customers line graph.

   - **Total Customers by Country Donut Chart**:

     - **Interaction Setting**: Configured so that this chart cross-filters the Total Customers by Product donut chart.

   - **Product Detail Page**:

   - **Orders vs. Profitability Scatter Graph**:

     - **Interaction Setting**: Configured so that this scatter graph does not affect any other visuals on the page.

   - **Top 10 Products Table**:

     - **Interaction Setting**: Configured so that this table does not affect any other visuals on the page.

## 10. Create Metrics for Users Outside the Company Using SQL

Connect to the Postgres database server hosted on Microsoft Azure using Visual Studio Code (VSCode) and run SQL queries, leveraging the SQLTools extension.

- **Install SQLTools Extension in VSCode**:#

- **Installation**:

    - Open Visual Studio Code.

    - Install SQLTools.

- **Configure Connection to the Postgres Database**:

    - Choose **PostgreSQL** as the database driver.#

    - Enter the following connection details:

    - **HOST**: `powerbi-data-analytics-server.postgres.database.azure.com`

    - **PORT**: `5432`

    - **DATABASE**: `postgres`

    - **USER**: `your username`

    - **PASSWORD**: `your password`

    - **SSL Encryption**:

    - Ensure that **SSL Encryption** is set to **enabled** in the connection settings.



