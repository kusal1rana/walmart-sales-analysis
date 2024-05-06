# walmart-sales-analysis

create database walmart;

use walmart;

## creating duplicate free temporary order table using cte function
with cto as (select distinct * from dbo.[order1])
select * from cto;

##Replace blank cells with NULL in the Order Table

UPDATE dbo.order1
SET OrderID = case when OrderID = '' then null else OrderID end,
    OrderDate = CASE WHEN OrderDate = '' THEN NULL ELSE OrderDate END,
    ShipDate = CASE WHEN ShipDate = '' THEN NULL ELSE ShipDate END,
    ShipMode = CASE WHEN ShipMode = '' THEN NULL ELSE ShipMode END,
    CustomerID = CASE WHEN CustomerID = '' THEN NULL ELSE CustomerID END,
    CustomerName = CASE WHEN CustomerName = '' THEN NULL ELSE CustomerName END;

select * from dbo.order1;

# 

SELECT *
FROM order1
WHERE OrderDate IS NULL
OR ShipDate IS NULL
OR ShipMode IS NULL
OR CustomerID IS NULL
OR CustomerName IS NULL;

### ct1 stands for cte transaction
with Ct1 as (select distinct * from dbo.[transaction])
 select * from ct1;

 ## creating duplicate free temporary table of location using cte

 with ctl as ( select distinct * from dbo.[location ])
 select * from ctl;


 # delete the row which contain null value

 delete from  dbo.[transaction]
 where OrderId is null;

 # find any blank cell in transaction table

 SELECT *
FROM dbo.[transaction]
WHERE OrderID = '' 
   OR Category = '' 
   OR SubCategory = '' 
   OR ProductName = '' 
   OR Sales = '' 
   OR Quantity = '' 
   OR Discount = '' 
   OR Profit = '';


 with ctt as ( select distinct * from dbo.[transaction]) 
 select * from ctt;


 ### calculating top selling products and top profitable product
 
 SELECT top(10)
    ProductName,
    SUM(Sales) AS TotalSales,
    ROW_NUMBER() OVER (ORDER BY SUM(Sales) DESC) AS SalesRank
	from
    dbo.[transaction]
GROUP BY
    ProductName;

	SELECT top(10)
    ProductName,
    SUM(Profit) AS TotalProfit,
    ROW_NUMBER() OVER (ORDER BY SUM(Profit) DESC) AS ProfitRank
FROM
    dbo.[transaction]
GROUP BY
    ProductName;

	## this is the most effectve query to get most selling, profitable as well as revenue wise product.
	## its pending. to be continued.


	## category generating highest revenue and profit
	SELECT Top(5)
    Category,
    SUM(Sales) AS TotalSales,
    SUM(Profit) AS TotalProfit,
    RANK() OVER (ORDER BY SUM(Sales) DESC) AS SalesRank,
    RANK() OVER (ORDER BY SUM(Profit) DESC) AS ProfitRank
FROM
    dbo.[transaction]
GROUP BY
    Category;

	Select top(10) * from dbo.[transaction];

	### sub-category with highest demand. its demand always represent quantity.

	SELECT top(5)
    SubCategory,
    SUM(Quantity) AS TotalQuantity,
    RANK() OVER (ORDER BY SUM(Quantity) DESC) AS DemandRank
FROM
    dbo.[transaction]
GROUP BY
    SubCategory;


	## average profit margin for each product category

	SELECT 
    Category,
    AVG(Profit / Sales) AS AvgProfitMargin,
    ROW_NUMBER() OVER (PARTITION BY Category ORDER BY AVG(Profit / Sales) DESC) AS MarginRank
FROM
    dbo.[transaction]
GROUP BY
    Category;


	## impact of discount on sales 
	
	SELECT
    Discount,
    AVG(Sales) AS AvgSales,
    ROW_NUMBER() OVER (ORDER BY AVG(Sales) DESC) AS SalesRank
FROM
    dbo.[transaction]
GROUP BY
    Discount;

	 ## another method   impact of discount on sales.

WITH TransactionSummary AS (
    SELECT
        t.Discount,
        t.ProductName,
        AVG(t.Sales) AS AvgSales,
        ROW_NUMBER() OVER (ORDER BY AVG(t.Sales) DESC) AS SalesRank
    FROM
       dbo.[transaction] t
    LEFT JOIN
        dbo.[order1] OI ON t.OrderID = OI.OrderID
    GROUP BY
        t.Discount, t.ProductName
)
SELECT TOP(10)
    Discount,
    ProductName,
    AvgSales,
    SalesRank
FROM
    TransactionSummary
ORDER BY
    AvgSales DESC;


## geographical analysis
## region and place generating most sales and profits
  ## first region wise
SELECT
    l.Region,
    SUM(t.Sales) AS TotalSales,
    SUM(t.Profit) AS TotalProfit,
    RANK() OVER (ORDER BY SUM(t.Sales) DESC) AS SalesRank,
    RANK() OVER (ORDER BY SUM(t.Profit) DESC) AS ProfitRank
FROM
   dbo.[location ] as l
   left join
   dbo.[transaction] as t on
    t.OrderID =l.OrderID
GROUP BY
    l.Region;


	## city wise 
	SELECT
    l.City,
    SUM(t.Sales) AS TotalSales,
    SUM(t.Profit) AS TotalProfit,
    RANK() OVER (ORDER BY SUM(t.Sales) DESC) AS SalesRank,
    RANK() OVER (ORDER BY SUM(t.Profit) DESC) AS ProfitRank
FROM
   dbo.[location ] as l
   left join
   dbo.[transaction] as t on
    t.OrderID =l.OrderID
GROUP BY
    l.City;

	### time series analysis

	SELECT
    DATEPART(YEAR, OrderDate) AS Year,
    DATEPART(MONTH, OrderDate) AS Month,
    SUM(Sales) AS TotalSales,
    RANK() OVER (PARTITION BY DATEPART(YEAR, OrderDate) ORDER BY SUM(Sales) DESC) AS SalesRank
FROM
    dbo.[order1] as o
	left join 
	dbo.[transaction]as t
	on o.OrderID = t.OrderID
GROUP BY
    DATEPART(YEAR, OrderDate),
    DATEPART(MONTH, OrderDate);


	## segment analysis for tailoring marketing, promotions and consumer segments




	SELECT count(shipmode),
    AVG(CASE WHEN ShipMode = 'Standard Class' THEN 1 ELSE 0 END) AS StandardClassUsage,
    ROW_NUMBER() OVER (ORDER BY AVG(CASE WHEN ShipMode = 'Standard Class' THEN 1 ELSE 0 END) DESC) AS SegmentRank
FROM
    dbo.[order1]
GROUP BY
    shipmode;


