# Retail Orders Data Analysis

## Project Overview
This project focuses on extracting, processing, and analyzing retail orders data to derive meaningful business insights. The dataset is sourced from Kaggle and is loaded into a Microsoft SQL Server database for further analysis using SQL queries.

## Dataset
**Source:** [Kaggle Retail Orders Dataset](https://www.kaggle.com/ankitbansal06/retail-orders)  
**File Used:** `orders.csv`

## Project Workflow
1. **Data Extraction & Preparation**
   - Download the dataset from Kaggle.
   - Extract the necessary CSV file from the ZIP archive.
   - Load the dataset into a Pandas DataFrame.
   - Clean and preprocess the data by handling missing values and renaming columns for consistency.
   - Compute new columns such as `discount`, `sale_price`, and `profit`.
   - Convert `order_date` to a datetime format.
   - Drop unnecessary columns.

2. **Data Loading into SQL Server**
   - Establish a connection to Microsoft SQL Server using SQLAlchemy.
   - Load the processed DataFrame into SQL Server using the `to_sql` method with the `append` option.

3. **Data Analysis with SQL Queries**
   - Identify the top 10 highest revenue-generating products.
   - Determine the top 5 highest-selling products in each region.
   - Analyze month-over-month growth comparisons for 2022 and 2023 sales.
   - Identify the highest sales month for each product category.
   - Determine which sub-category had the highest profit growth in 2023 compared to 2022.

## Code Implementation
### Data Extraction and Preprocessing
```python
import kaggle
import zipfile
import pandas as pd
import sqlalchemy as sal

# Download dataset
!kaggle datasets download ankitbansal06/retail-orders -f orders.csv

# Extract CSV from ZIP file
zip_ref = zipfile.ZipFile('orders.csv.zip')
zip_ref.extractall()
zip_ref.close()

# Load dataset into Pandas
df = pd.read_csv('orders.csv', na_values=['Not Available', 'unknown'])

# Standardize column names
df.columns = df.columns.str.lower().str.replace(' ', '_')

# Compute additional columns
df['discount'] = df['list_price'] * df['discount_percent'] * 0.01
df['sale_price'] = df['list_price'] - df['discount']
df['profit'] = df['sale_price'] - df['cost_price']
df['order_date'] = pd.to_datetime(df['order_date'], format="%Y-%m-%d")

# Drop unnecessary columns
df.drop(columns=['list_price', 'cost_price', 'discount_percent'], inplace=True)

# Load data into SQL Server
engine = sal.create_engine('mssql://LAPTOP-7BAE2NC2/master?driver=ODBC+DRIVER+17+FOR+SQL+SERVER')
conn = engine.connect()
df.to_sql('df_orders', con=conn, index=False, if_exists='append')
```

### SQL Queries for Analysis
#### Top 10 Highest Revenue Generating Products
```sql
SELECT TOP 10 product_id, SUM(sale_price) AS sales
FROM df_orders
GROUP BY product_id
ORDER BY sales DESC;
```

#### Top 5 Highest Selling Products in Each Region
```sql
WITH cte AS (
    SELECT region, product_id, SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY region, product_id
)
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER(PARTITION BY region ORDER BY sales DESC) AS rn
    FROM cte
) A
WHERE rn <= 5;
```

#### Month-over-Month Growth Comparison (2022 vs 2023)
```sql
WITH cte AS (
    SELECT YEAR(order_date) AS order_year, MONTH(order_date) AS order_month, SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY YEAR(order_date), MONTH(order_date)
)
SELECT order_month,
       SUM(CASE WHEN order_year=2022 THEN sales ELSE 0 END) AS sales_2022,
       SUM(CASE WHEN order_year=2023 THEN sales ELSE 0 END) AS sales_2023
FROM cte
GROUP BY order_month
ORDER BY order_month;
```

#### Highest Sales Month for Each Product Category
```sql
WITH cte AS (
    SELECT category, FORMAT(order_date, 'yyyyMM') AS order_year_month, SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY category, FORMAT(order_date, 'yyyyMM')
)
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER(PARTITION BY category ORDER BY sales DESC) AS rn
    FROM cte
) A
WHERE rn = 1;
```

#### Sub-Category with Highest Profit Growth (2023 vs 2022)
```sql
WITH cte AS (
    SELECT sub_category, YEAR(order_date) AS order_year, SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY sub_category, YEAR(order_date)
),
cte2 AS (
    SELECT sub_category,
           SUM(CASE WHEN order_year=2022 THEN sales ELSE 0 END) AS sales_2022,
           SUM(CASE WHEN order_year=2023 THEN sales ELSE 0 END) AS sales_2023
    FROM cte
    GROUP BY sub_category
)
SELECT TOP 1 *, (sales_2023 - sales_2022) AS growth
FROM cte2
ORDER BY growth DESC;
```

## Conclusion
This project demonstrates how to:
- Extract and preprocess retail orders data.
- Load the data into SQL Server for efficient querying.
- Analyze sales trends and business performance using SQL queries.

By leveraging SQL and Python, this analysis provides valuable insights for retail decision-making, such as identifying top-selling products, tracking revenue growth, and understanding category-wise sales performance.

## Future Enhancements
- Automate data extraction and transformation with ETL pipelines.
- Create interactive dashboards using Power BI or Tableau.
- Perform predictive analytics to forecast sales trends.

## Author
[Your Name]  
[Your Contact Information]  
[Your LinkedIn Profile]

