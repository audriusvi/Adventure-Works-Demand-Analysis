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

## Findings
When seeking an unusual distribution, a noteworthy situation arises in the offline sales of Canada and the USA for the year 2002. Canada and the USA were and remained (by 2005) the most important sales group, accounting for more than 70 percent of total sales.

![image](https://github.com/audriusvi/Adventure-Works-Demand-Analysis/assets/168005242/bc44783e-8c3e-4307-9725-d42be7e05349)


The profit column for June 2002 appears anomalous. While the average monthly profit stands at $345,000, June sees a loss of $29,000.

![image](https://github.com/audriusvi/Adventure-Works-Demand-Analysis/assets/168005242/61340f1c-f5db-4694-9da9-2decf5b0fc74)


A deeper look reveals that it occurred due to Mountain Bikes sales that led to 254K loss. In fact, it was all bike model Mountain-100. It was sold in 2 colors and 4 different frame sizes. We see that losses incurred in all cases.

![image](https://github.com/audriusvi/Adventure-Works-Demand-Analysis/assets/168005242/93d907d5-f8be-4c82-971d-9e226e4fc3fa)



The Mountain-100 model began sales in July 2001, coinciding with the launch of Adventure Works in the same month. Sales of the product ceased at the end of June 2002, after twelve months. Why did the sales stop? The primary reason was the introduction of two new Mountain bike models, the Mountain-200 and Mountain-300. Suddenly, the Mountain-100 became obsolete, leading to the decision to discontinue the product and conduct a clearance sale. All remaining units of this model were sold at a 35% discount, resulting in a considerable loss for that month.

![image](https://github.com/audriusvi/Adventure-Works-Demand-Analysis/assets/168005242/bedc3b2b-fa1a-4ba6-8ed6-34730b75c83f)



Let's take a look at the sales performance trend in mountain bikes over the last three months. On the y-axis, you can see the number of products sold, while the x-axis represents the profit or loss of total sales during this period. Specifically, we are focusing on the Mountain-500 bike model.

It's interesting to note that the same model, but with different variations, resulted in different profit situations. In other words, while black bikes met production expectations, silver bikes were overstocked, leading to sales at a loss.

This illustrates the importance of identifying trends over time and understanding customer preferences

![image](https://github.com/audriusvi/Adventure-Works-Demand-Analysis/assets/168005242/dd0d1706-2fa2-475d-8c5c-157d99aedf19)



What's currently trending, and what production decisions should we make, particularly regarding color?

For most bike colors, sales correlate with overall bike sales. However, an interesting trend emerges in the sales of yellow and red bikes. Historically, we sold more red bikes than yellow, but this changed in June 2003.

Between June 2002 and June 2003, we sold over 10,000 red bikes, constituting 32% of all sales, compared to 3,400 yellow bikes, which accounted for 10% of all sales.

The following year, red bike sales dropped to just 1% of all sales, while yellow bike sales increased to 33% of all sales. It's evident that yellow is trending.

![image](https://github.com/audriusvi/Adventure-Works-Demand-Analysis/assets/168005242/59728e3f-254a-4d59-afcf-42c91f7395bf)



But what about red color? Should we introduce new models in red, or not? Let's consider another angle: the preferences of one of our customer groups, children.

In 2001, the University of Taiwan conducted research with the main purpose of understanding children’s color preferences. The aim was to create attractive pictures for books, thereby increasing children's interest in reading. The research revealed that the most popular color among children is red, followed by blue and yellow.

On the right, you can see a possible selection of children's frame sizes for bikes at Adventure Works. We observe that clients are currently able to order silver, black, and yellow bikes. While yellow bikes align with children's expectations, black bikes do as well, at least partially.

However, it would be a great idea to introduce children's bikes in red and blue to the market, especially considering that changing the color is less technically complex than altering other bike specifications.

![image](https://github.com/audriusvi/Adventure-Works-Demand-Analysis/assets/168005242/aa5d48f3-9805-4fbb-b72a-961ae5bf79b6)


What we can improve?
The current selection of children's bikes is inadequate for the market; we should offer red and blue bikes.

Feedback is crucial for several reasons. Customer preferences are constantly evolving, and feedback helps us stay adaptable by providing insights into changing market trends and customer behaviors. However, we have very few reviews in our database, so it's essential that we work on this.

Adventure Works boasts 290 active employees, indicating a substantial workforce. With such diversity among our colleagues, each bringing various opinions and perspectives, they can contribute significantly to product selection and initial testing before products go into production.
