# Adventure Works Demand Analysis
## The aim of the project
To create an interactive sales dashboard for fictional Adventure Works Bicycle Shop, analyze product demand, and make a presentation of findings.
## Tools used
Big Query, Tableau, Google Spreadsheets.
## Data Processing
Adventure Works database of 2001-2004 years is used in this analysis. SQL code snippet to get customer type.

``` sql
-- Data extraction from AdventureWorks database of 2001-2004 years.
-- Used for data analysis, dashboard (Tableau Public) and presentations for Executive Leadership and Operations team.
-- Returns a table of customer type.

SELECT
  CustomerID AS customer_id,
  CustomerType AS customer_type
FROM
  tc-da-1.adwentureworks_db.customer;
```

Table of all orders of the shop.

``` sql
-- Data extraction from AdventureWorks database of 2001-2004 years.
-- Used for data analysis, dashboard (Tableau Public) and presentations for Executive Leadership and Operations team.
-- Returns a table of all orders.
-- sub_total - without taxes, due_toal - with taxes and shipping, line_total - from Salesorderdetail table.

WITH
-- order_line_total is usded for product due total calculation in Tableau
  order_ltotal AS (
  SELECT
    SalesOrderID,
    SUM(LineTotal) AS order_line_total
  FROM
    tc-da-1.adwentureworks_db.salesorderdetail
  GROUP BY
    SalesOrderID)

SELECT
  SalesOrderID AS order_id,
  CustomerID AS customer_id,
  SalesPersonID AS sales_person_id,
  TerritoryID AS territory_id,
  SubTotal AS sub_total,
  TotalDue AS due_total,
  order_line_total AS line_total
FROM
  tc-da-1.adwentureworks_db.salesorderheader
LEFT JOIN
  order_ltotal
USING
  (SalesOrderID)
```

All ordered products.

``` sql
-- Data extraction from AdventureWorks database of 2001-2004 years.
-- Used for data analysis, dashboard (Tableau Public) and presentations for Executive Leadership and Operations team.
-- Returns a table of all ordered products.
-- order_detail_id - unique number per product sold (PK), order_id - unique number per order.

WITH
  brief_order_detail AS (
  SELECT
    SalesOrderID,
    SalesOrderDetailID,
    OrderQty,
    ProductID,
    UnitPrice,
    UnitPriceDiscount,
    LineTotal,
    ModifiedDate
  FROM
    tc-da-1.adwentureworks_db.salesorderdetail),

  brief_product AS (
  SELECT
    ProductID,
    Name,
    StandardCost,
    ListPrice,
    ProductSubcategoryID,
    SellStartDate
  FROM
    tc-da-1.adwentureworks_db.product),

  brief_subcategory AS (
  SELECT
    ProductSubcategoryID,
    ProductCategoryID,
    Name
  FROM
    tc-da-1.adwentureworks_db.productsubcategory),

  brief_category AS (
  SELECT
    ProductCategoryID,
    Name
  FROM
    tc-da-1.adwentureworks_db.productcategory)

SELECT
  brief_order_detail.SalesOrderDetailID AS order_detail_id,
  brief_order_detail.SalesOrderID AS order_id,
  brief_order_detail.OrderQty AS order_qty,
  brief_order_detail.ProductID AS product_id,
  brief_order_detail.UnitPrice AS unit_price,
  brief_order_detail.UnitPriceDiscount AS unit_price_discount,
  brief_order_detail.LineTotal AS line_total,
  brief_order_detail.ModifiedDate AS modified_date,
  brief_product.Name AS product,
  brief_product.StandardCost AS standard_cost,
  brief_product.ListPrice AS list_price,
  brief_subcategory.Name AS subcategory,
  brief_category.Name AS category
FROM
  brief_order_detail
JOIN
  brief_product
ON
  brief_order_detail.ProductID = brief_product.ProductID
  AND DATE(brief_order_detail.ModifiedDate) >= brief_product.SellStartDate
LEFT JOIN
  brief_subcategory
ON
  brief_product.ProductSubcategoryID = brief_subcategory.ProductSubcategoryID
LEFT JOIN
  brief_category
ON
  brief_subcategory.ProductCategoryID = brief_category.ProductCategoryID;
```

Sales territories.

``` sql
-- Data extraction from AdventureWorks database of 2001-2004 years.
-- Used for data analysis, dashboard (Tableau Public) and presentations for Executive Leadership and Operations team.
-- Returns sales territories table.

SELECT
  TerritoryID AS territory_id,
  Name AS territory,
  CountryRegionCode AS country_code,
  CASE
    WHEN CountryRegionCode = 'US' THEN 'USA'
    ELSE Name
  END AS country,
  territory.Group AS tr_group
FROM
  tc-da-1.adwentureworks_db.salesterritory AS territory
ORDER BY
  territory_id;
```

## Sales Overview Dashboard
The processed and cleaned data is utilized to create the Sales Overview Dashboard in the Tableau Public environment.

[Interactive dashboard can be accessed here.](https://public.tableau.com/views/M2S2_project/Dashboard?:language=en-US&:sid=&:display_count=n&:origin=viz_share_link)

![image](https://github.com/audriusvi/Adventure-Works-Demand-Analysis/assets/168005242/f502ff43-91b0-4372-81eb-f28af67e9fd1)
