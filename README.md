![1711408967531](https://github.com/user-attachments/assets/58453876-951e-4fdd-9cb4-907d6e9dae7a)

# 📊 Project Title: AdventureWorks Dataset Exploring  
🤵 Author: Tri Nguyen  <br> 
📆 Date: Nov. 10, 2024 <br> 
💻 Tools Used: SQL - BigQuery Platform 

---

## 📑 Table of Contents  
1. [📌 Background & Overview](#-background--overview)  
2. [📂 Dataset Description & Data Structure](#-dataset-description--data-structure)  
3. [🧠 Problem Solving Process](#-problem-solving-process)  
4. [📊 Explore the Dataset & Generate Insights](#-explore-the-dataset--generate_insights)  
5. [🔎 Final Conclusion & Recommendations](#-final-conclusion--recommendations)

---

## 📌 Background & Overview  

### Objective:
### 📖 What is this project about? 
 
> This project utilizes advanced SQL techniques to analyze the AdventureWorks dataset, revealing trends, enhancing data visualization, and supporting decision-making within a business environment.  

### 👤 Who is this project for?  


➡️ **Sales manager** who want to understand the Sales revenue components and Sales trend. 

###  ❓Business Questions:  

- Analyze data on items, sales, order quantities, growth rates, top categories, top territories, and total discount costs by subcategories.
- Assess customer retention rates, stock level trends, month-over-month differences, and stock-to-sales ratios.
- Evaluate the number and value of pending orders in 2014.

### 🎯Project Outcome:  

- **Sales and Growth Trends**: Bike Racks and Road Frames achieved the highest sales revenue and quantity, respectively. Mountain Frames and Socks showed notable year-over-year growth.
- **Discounts and Customer Retention**: Helmets had continuous discount promotions with increased discount costs from 2012 to 2013, while customer retention dropped significantly after the first purchase, indicating the need for better retention strategies.
- **Stock and Order Management**: Quarterly stock levels consistently declined, indicating robust sales. A substantial number of pending orders highlight the need to assess vendor performance for efficiency and suggest improvements to their order processes.

---

## 📂 Dataset Description & Data Structure  

### 📌 Data Source  
- Source: The AdventureWorks2019 dataset by Microsoft is a comprehensive sample database that simulates a manufacturing company's operations.
- Size: Over 121000 rows 
- Format: 
  <details>
  <summary>To access the dataset, follow these step</summary>
  
  - Log in to your Google Cloud Platform account and create a new project.
  - Navigate to the BigQuery console and select your newly created project.
  - In the navigation panel, select "Add Data" and then choose "Star a project by name".
  - Enter the project name **"adventureworks2019"** and click "Enter".
  - Click on the **"adventureworks2019"** table to open it.

  </details>

### 📊 Data Structure & Relationships  

#### 1️⃣ Tables Used:  
There're **6 tables** were used in this project  

#### 2️⃣ Table Schema & Data Snapshot  

Table 1: Sales.SalesOrderHeader  

| Name                   | Data type       | Description / Attributes                                                                          |
|------------------------|-----------------|---------------------------------------------------------------------------------------------------|
| SalesOrderID           | int             | Primary key. Identity / Auto increment column                                            |
| RevisionNumber         | tinyint         | Incremental number to track changes to the sales order over time. Default: 0                      |
| OrderDate              | datetime        | Dates the sales order was created. Default: getdate()                                             |
| DueDate                | datetime        | Date the order is due to the customer.                                                            |
| ShipDate               | datetime        | Date the order was shipped to the customer.                                                       |
| Status                 | tinyint         | Order current status. 1 = In process; 2 = Approved; 3 = Backordered; 4 = Rejected; 5 = Shipped; 6 = Cancelled Default: 1 |
| OnlineOrderFlag        | bit             | 0 = Order placed by sales person. 1 = Order placed online by customer. Default: 1                 |
| SalesOrderNumber       | nvarchar(25)    | Unique sales order identification number. Computed: isnul(N'SO'+CONVERT(nvarchar(23),[SalesOrderID]),N'*** ERROR ***') |
| PurchaseOrderNumber    | nvarchar(25)    | Customer purchase order number reference.                                                         |
| AccountNumber          | nvarchar(15)    | Financial accounting number reference.                                                            |
| CustomerID             | int             | Customer identification number. Foreign key to Customer.BusinessEntityID.                         |
| SalesPersonID          | int             | Sales person who created the sales order. Foreign key to SalesPerson.BusinessEntityID.            |
| TerritoryID            | int             | Territory in which the sale was made. Foreign key to SalesTerritory.SalesTerritoryID.             |
| BillToAddressID        | int             | Customer billing address. Foreign key to Address.AddressID.                                       |
| ShipToAddressID        | int             | Customer shipping address. Foreign key to Address.AddressID.                                      |
| ShipMethodID           | int             | Shipping method. Foreign key to ShipMethod.ShipMethodID.                                          |
| CreditCardID           | int             | Credit card identification number. Foreign key to CreditCard.CreditCardID.                        |
| CreditCardApprovalCode | varchar(15)     | Approval code provided by the credit card company.                                                |
| CurrencyRateID         | int             | Currency exchange rate used. Foreign key to CurrencyRate.CurrencyRateID.                          |
| SubTotal               | money           | Sales subtotal. Computed as SUM(SalesOrderDetail.LineTotal) for the appropriate SalesOrderID. Default: 0.00 |
| TaxAmt                 | money           | Tax amount. Default: 0.00                                                                         |
| Freight                | money           | Shipping cost. Default: 0.00                                                                      |
| TotalDue               | money           | Total due from customer. Computed as Subtotal + TaxAmt + Freight. Computed: isnul(((SubTotal)+(TaxAmt))+(Freight),(0)) |
| Comment                | nvarchar(128)   | Sales representative comments.                                                                    |
| rowguid                | uniqueidentifier| ROWGUIDCOL number uniquely identifying the record. Used to support a merge replication sample. Default: newid() |
| ModifiedDate           | datetime       | Date and time the record was last updated. Default: getdate()                                     |


Table 2: Sales.SalesOrderDetail  

| Column Name            | Data Type       | Description/Attributes                                                                   |
|------------------------|-----------------|------------------------------------------------------------------------------------------|
| SalesOrderID           | int             | Primary key. Foreign key to SalesOrderHeader.SalesOrderID.                                |
| SalesOrderDetailID     | int             | Primary key. One incremental unique number per product sold. Identity / Auto increment.   |
| CarrierTrackingNumber  | nvarchar(25)    | Shipment tracking number supplied by the shipper.                                         |
| OrderQty               | smallint        | Quantity ordered per product.                                                            |
| ProductID              | int             | Product sold to customer. Foreign key to Product.ProductID.                               |
| SpecialOfferID         | int             | Promotional code. Foreign key to SpecialOffer.SpecialOfferID.                             |
| UnitPrice              | money           | Selling price of a single product.                                                       |
| UnitPriceDiscount      | money           | Discount amount. Default: 0.0.                                                           |
| LineTotal              | numeric(38, 6)   | Per product subtotal. Computed as UnitPrice * (1 - UnitPriceDiscount) * OrderQty. Computed: isnull((([UnitPrice]*((1.0-[UnitPriceDiscount])*[OrderQty]),(0.0)) |
| rowguid                | uniqueidentifier | ROWGUIDCOL number uniquely identifying the record. Used to support a merge replication sample. Default: newid() |
| ModifiedDate           | datetime         | Date and time the record was last updated. Default: getdate()                                                 |


Table 3: Production.Product  

| Name                     | Data type        | Description / Attributes                                                                                      |
|--------------------------|------------------|---------------------------------------------------------------------------------------------------------------|
| ProductID                | int              | Primary key for Product records. Identity / Auto increment column                                             |
| Name                     | nvarchar(50)     | Name of the product.                                                                                          |
| ProductNumber            | nvarchar(25)     | Unique product identification number.                                                                         |
| MakeFlag                 | bit              | 0 = Product is purchased, 1 = Product is manufactured in-house. Default: 1                                    |
| FinishedGoodsFlag        | bit              | 0 = Product is not a salable item. 1 = Product is salable. Default: 1                                         |
| Color                    | nvarchar(15)     | Product color.                                                                                                |
| SafetyStockLevel         | smallint         | Minimum inventory quantity.                                                                                   |
| ReorderPoint             | smallint         | Inventory level that triggers a purchase order or work order.                                                 |
| StandardCost             | money            | Standard cost of the product.                                                                                 |
| ListPrice                | money            | Selling price.                                                                                                |
| Size                     | nvarchar(5)      | Product size.                                                                                                 |
| SizeUnitMeasureCode      | nchar(3)         | Unit of measure for Size column.                                                                              |
| WeightUnitMeasureCode    | nchar(3)         | Unit of measure for Weight column.                                                                            |
| Weight                   | decimal(8, 2)    | Product weight.                                                                                               |
| DaysToManufacture        | int              | Number of days required to manufacture the product.                                                           |
| ProductLine              | nchar(2)         | R = Road, M = Mountain, T = Touring, S = Standard                                                             |
| Class                    | nchar(2)         | H = High, M = Medium, L = Low                                                                                 |
| Style                    | nchar(2)         | W = Womens, M = Mens, U = Universal                                                                           |
| ProductSubcategoryID     | int              | Product is a member of this product subcategory. Foreign key to ProductSubCategory.ProductSubCategoryID.       |
| ProductModelID           | int              | Product is a member of this product model. Foreign key to ProductModel.ProductModelID.                         |
| SellStartDate            | datetime         | Date the product was available for sale.                                                                      |
| SellEndDate              | datetime         | Date the product was no longer available for sale.                                                            |
| DiscontinuedDate         | datetime         | Date the product was discontinued.                                                                            |
| rowguid                  | uniqueidentifier | ROWGUIDCOL number uniquely identifying the record. Used to support a merge replication sample. Default: newid()|
| ModifiedDate             | datetime         | Date and time the record was last updated. Default: getdate()                                                 |


Table 4: Production.ProductSubcategory  

| Name                  | Data type        | Description / Attributes                                                                                      |
|-----------------------|------------------|---------------------------------------------------------------------------------------------------------------|
| ProductSubcategoryID  | int              | Primary key for ProductSubcategory records. Identity / Auto increment column.                                  |
| ProductCategoryID     | int              | Product category identification number. Foreign key to ProductCategory.ProductCategoryID.                      |
| Name                  | nvarchar(50)     | Subcategory description.                                                                                       |
| rowguid               | uniqueidentifier | ROWGUIDCOL number uniquely identifying the record. Used to support a merge replication sample. Default: newid()|
| ModifiedDate          | datetime         | Date and time the record was last updated. Default: getdate()                                                 |


Table 5: Production.WorkOrder  

| Name                  | Data type        | Description / Attributes                                                                                      |
|-----------------------|------------------|---------------------------------------------------------------------------------------------------------------|
| WorkOrderID           | int              | Primary key for WorkOrder records. Identity / Auto increment column                                           |
| ProductID             | int              | Product identification number. Foreign key to Product.ProductID                                               |
| OrderQty              | int              | Product quantity to build                                                                                     |
| StockedQty            | int              | Quantity built and put in inventory. Computed: isnull([OrderQty] - [ScrappedQty], 0)                           |
| ScrappedQty           | smallint         | Quantity that failed inspection                                                                               |
| StartDate             | datetime         | Work order start date                                                                                         |
| EndDate               | datetime         | Work order end date                                                                                           |
| DueDate               | datetime         | Work order due date                                                                                           |
| ScrapReasonID         | smallint         | Reason for inspection failure                                                                                 |
| ModifiedDate          | datetime         | Date and time the record was last updated. Default: getdate()                                                 |


Table 6: Purchasing.PurchaseOrderHeader  

| Name                | Data type  | Description / Attributes                                                                                              |
|---------------------|------------|-----------------------------------------------------------------------------------------------------------------------|
| PurchaseOrderID     | int        | Primary key. Identity / Auto increment column.                                                                         |
| RevisionNumber      | tinyint    | Incremental number to track changes to the purchase order over time. Default: 0                                        |
| Status              | tinyint    | Order current status. 1 = Pending; 2 = Approved; 3 = Rejected; 4 = Complete. Default: 1                               |
| EmployeeID          | int        | Employee who created the purchase order. Foreign key to Employee.BusinessEntityID.                             |
| VendorID            | int        | Vendor with whom the purchase order is placed. Foreign key to Vendor.BusinessEntityID.                        |
| ShipMethodID        | int        | Shipping method. Foreign key to ShipMethod.ShipMethodID.                                                      |
| OrderDate           | datetime   | Purchase order creation date. Default: getdate()                                                              |
| ShipDate            | datetime   | Estimated shipment date from the vendor.                                                                      |
| SubTotal            | money      | Purchase order subtotal. Computed as SUM(PurchaseOrderDetail.LineTotal) for the appropriate PurchaseOrderID. Default: 0.00 |
| TaxAmt              | money      | Tax amount. Default: 0.00                                                                                      |
| Freight             | money      | Shipping cost. Default: 0.00                                                                                   |
| TotalDue            | money      | Total due to vendor. Computed as Subtotal + TaxAmt + Freight. Computed: isnull((([SubTotal] + [TaxAmt]) + [Freight]), (0)) |
| ModifiedDate        | datetime   | Date and time the record was last updated. Default: getdate()                                                  |


#### 3️⃣ Data Relationships:  

- Field `SalesOrderDetai.SalesOrderID` is foreign key to `SalesOrderHeader.SalesOrderID`
- Field `SalesOrderDetai.ProductID` is foreign key to `Product.ProductID`
- Field `Product.ProductID` is foreign key to `WorkOrder.ProductID`
- Field `Product.ProductSubcategoryID` is foreign key to `ProductSubCategory.ProductSubcategoryID`

![image](https://github.com/user-attachments/assets/9ddad757-f722-4866-8cfe-d3921379c990)

---

## 🧠 Problem Solving Process  

| 1️⃣ Understand Problem | 2️⃣ Break it down into smaller pieces | 3️⃣ Ideate | 4️⃣ Implement and Review 
|-|-|-|-
| Which to be calculated (sum, count, ratio, etc.) and grouped? <br> Does it needs any filter (time, conditions, etc.)? | Which tables have data that I want to get? <br> Which columns have data corresponded to the problem? <br> Can I get the data I by one step, if not, break down even smaller? | How many step did I need to get the final result? <br> Is it optimized? | _We'll go through this step in the order of each query listed below_ <br> ⬇️

---

## 📊 Explore the Dataset & Generate Insights

#### Query 1️⃣: Calc Quantity of items, Sales value & Order quantity by each Subcategory in Last 12 Mth
```sql
SELECT 
  format_datetime('%b %Y', sod.ModifiedDate) as period
  ,pps.Name as name
  ,sum(sod.OrderQty) as qty_item
  ,round(sum(sod.LineTotal),2) as total_sales
  ,count(distinct sod.SalesOrderID) as order_cnt
FROM `adventureworks2019.Sales.SalesOrderDetail` as sod
left join `adventureworks2019.Production.Product` as pp 
  on sod.ProductID = pp.ProductID
left join `adventureworks2019.Production.ProductSubcategory` as pps 
  on cast(pp.ProductSubcategoryID as int) = pps.ProductSubcategoryID
where date(sod.ModifiedDate) >= (select date_add(max(date(ModifiedDate)), interval -12 month)
                             from `adventureworks2019.Sales.SalesOrderDetail`)
group by 1,2
order by 2,1 desc;
```
|Row	|period|name|qty_item|total_sales|order_cnt|
|---|---|---|---|---|---
|1	|Jun 2013|Bib-Shorts|2|116.99|1|
|2	|Jul 2013|Bib-Shorts|2|116.99|1|
|3	|Feb 2014|Bib-Shorts|4|233.97|2|
|4	|Apr 2014|Bib-Shorts|4|233.97|1|
|5	|Sep 2013|Bike Racks|312|22828.51|71|
|6	|Oct 2013|Bike Racks|284|21181.2|70|
|7	|Nov 2013|Bike Racks|142|11472.0|50|
|8  |	...

💡 Bike Racks have had the highest sales revenue in the past 12 months, indicating strong market demand.

#### Query 2️⃣: Calc % YoY growth rate by SubCategory & release top 3 cat with highest grow rate. Can use metric: quantity_item. Round results to 2 decimal
```sql
with qty_data as (
    select 
      pps.Name as name
      ,format_date("%Y", sod.ModifiedDate) as year 
      ,sum(sod.OrderQty) as qty_item
    from `adventureworks2019.Sales.SalesOrderDetail` as sod
    left join `adventureworks2019.Production.Product` as pp on sod.ProductID = pp.ProductID
    left join `adventureworks2019.Production.ProductSubcategory` as pps on cast(pp.ProductSubcategoryID as int) = pps.ProductSubcategoryID
    group by 1,2
  ),
  sale_diff as (
    select *
      ,lag(qty_item) over (partition by name order by year) as prv_qty
      ,round((qty_item - lag(qty_item) over (partition by name order by year))/(lag(qty_item) over (partition by name order by year)),2) as qty_diff
    from qty_data 
  ),
  sale_rk as (
    select *
      ,dense_rank() over (order by qty_diff desc) as dkr 
    from sale_diff 
  )
select distinct 
  name
  ,qty_item 
  ,prv_qty
  ,qty_diff
  ,dkr 
from sale_rk 
where dkr <= 3 
order by dkr;
```
|Row	|name|qty_item|prv_qty|qty_diff|dkr
|---|---|---|---|---|---
|1	|Mountain Frames|3168|510|5.21|1
|2	|Socks|2724|523|4.21|2
|3	|Road Frames|5564|1137|3.89|3

💡 Road Frames had the highest quantity sales, while Mountain Frames and Socks showed the highest _year-over-year_ growth rates.

#### Query 3️⃣: Query 3: Ranking Top 3 TeritoryID with biggest Order quantity of every year. If there's TerritoryID with same quantity in a year, do not skip the rank number
```sql
with 
  data as (
    select 
      format_date("%Y", sod.ModifiedDate) as year 
      ,soh.TerritoryID
      ,sum(sod.OrderQty) as order_cnt
    from `adventureworks2019.Sales.SalesOrderDetail` as sod
    left join `adventureworks2019.Sales.SalesOrderHeader` as soh on sod.SalesOrderID = soh.SalesOrderID
    group by 1,2 
    order by 1 desc 
  ),
  ranking as (
    select *
      ,dense_rank() over (partition by year order by order_cnt desc) as rk
    from data 
    order by 1 desc 
  )
select * 
from ranking 
where rk <=3;
```
|Row	|year|TerritoryID|order_cnt|rk
|-|-|-|-|-
|1	|2014|4|11632|1
|2	|2014|6|9711|2
|3	|2014|1|8823|3
|4	|2013|4|26682|1
|5	|2013|6|22553|2
|6	|2013|1|17452|3
|7	|2012|4|17553|1
|8	|2012|6|14412|2
|9	|2012|1|8537|3
|10	|2011|4|3238|1
|11	|2011|6|2705|2
|12	|2011|1|1964|3

💡 Territory No. 4 consistently had the highest order rates each year, with Territory No. 6 following closely behind.

#### Query 4️⃣: Query 4: Calc Total Discount Cost belongs to Seasonal Discount for each SubCategory
```sql
with discount_data as (
  select distinct 
    sod.ModifiedDate 
    ,ps.Name 
    ,so.Type
    ,so.DiscountPct*sod.UnitPrice*sod.OrderQty as dis_cost 
  from `adventureworks2019.Sales.SalesOrderDetail` as sod
  left join `adventureworks2019.Production.Product` as p  on sod.ProductID = p.ProductID
  left join `adventureworks2019.Production.ProductSubcategory` as ps on cast(p.ProductSubcategoryID as int) = ps.ProductSubcategoryID
  left join `adventureworks2019.Sales.SpecialOffer` as so on so.SpecialOfferID = sod.SpecialOfferID
  where lower(so.Type) like '%seasonal discount%'
  )
select 
  format_date("%Y", ModifiedDate) as year
  ,Name
  ,sum(dis_cost) as total_cost
from discount_data
group by 1,2
order by 2,1; 
```
|Row	|year|Name|total_cost
|-|-|-|-
|1	|2012|Helmets|149.71669
|2	|2013|Helmets|543.21975

💡 The "Helmets" sub-category was the only one with discount promotions in both 2012 and 2013. 
There was a significant increase in discount costs from 2012 to 2013, suggesting a more aggressive discount strategy or higher sales volumes benefiting from seasonal discounts.

#### Query 5️⃣: Query 5: Retention rate of Customer in 2014 with status of Successfully Shipped (Cohort Analysis)
```sql
with 
  info as (
    select 
      extract(year from ModifiedDate) as yr
      ,extract(month from ModifiedDate) as mth 
      ,CustomerID 
      ,count(distinct SalesOrderID) sale_cnt 
    from `adventureworks2019.Sales.SalesOrderHeader`
    where extract(year from ModifiedDate) = 2014 and Status = 5
    group by 1,2,3
  )
  ,row_num as (
    select 
      *
      ,row_number() over (partition by CustomerID order by mth) as row_nb
    from info 
    order by CustomerID, mth 
  )
  , first_order as (
    select distinct
      mth
      ,yr
      ,CustomerID
    from row_num
    where row_nb = 1
  )
  , all_join as (
    select distinct 
      a.mth as mth_order
      ,a.yr
      ,a.CustomerID
      ,b.mth as mth_join
      ,concat('M - ',a.mth - b.mth) as mth_diff
    from info as a 
    left join first_order as b on a.CustomerID = b.CustomerID
    order by 3
  )
select
  mth_order, mth_diff, count(CustomerID) as customer_cnt
from all_join 
group by 1,2 
order by 1,2;
```
|Row	|mth_order|mth_diff|customer_cnt
|-|-|-|-
|1	|1|M - 0|2076
|2	|2|M - 0|1805
|3	|2|M - 1|78
|4	|3|M - 0|1918
|5	|3|M - 1|51
|6	|3|M - 2|89
|7	|4|M - 0|1906
|8	|4|M - 1|43
|9	|4|M - 2|61
|10	|4|M - 3|252
|11	|5|M - 0|1947
|12	|5|M - 1|34
|13	|5|M - 2|58
|14	|5|M - 3|234
|15	|5|M - 4|96
|16	|6|M - 0|909
|17	|6|M - 1|40
|18	|6|M - 2|44
|19	|6|M - 3|44
|20	|6|M - 4|58
|21	|6|M - 5|61
|22	|7|M - 0|148
|23	|7|M - 1|10
|24	|7|M - 2|7
|25	|7|M - 3|7
|26	|7|M - 4|11
|27	|7|M - 5|8
|28	|7|M - 6|18

💡 Retention drops significantly from the first month to subsequent months, with few customers returning after their initial purchase. This trend is consistent across all months analyzed, highlighting **the need for improved customer retention strategies**.

#### Query 6️⃣: Trend of Stock level & MoM diff % by all product in 2011. If %gr rate is null then 0. Round to 1 decimal
```sql
with data_2011 as (
  select 
    p.Name
    ,extract(month from wo.ModifiedDate) as mth
    ,extract(year from wo.ModifiedDate) as yr 
    ,sum(StockedQty) as stock_crt
  from `adventureworks2019.Production.WorkOrder` as wo
  left join `adventureworks2019.Production.Product` as p on wo.ProductID = p.ProductID
  where FORMAT_TIMESTAMP("%Y", wo.ModifiedDate) = '2011'
  group by 1,2,3 
  order by 1,2 desc
)
select 
  Name
  ,mth ,yr
  ,stock_crt, stock_prv 
  ,round(coalesce((stock_crt/stock_prv - 1)*100,0),2) as diff_p
from (select * 
        ,lag(stock_crt,1) over (partition by name order by mth) as stock_prv
      from data_2011 ) 
order by 1,2 desc;
```
|Row	|Name|mth|yr|stock_crt|stock_prv|diff_p
|-|-|-|-|-|-|-
|1	|BB Ball Bearing|12|2011|8475|14544|-41.73
|2	|BB Ball Bearing|11|2011|14544|19175|-24.15
|3	|BB Ball Bearing|10|2011|19175|8845|116.79
|4	|BB Ball Bearing|9|2011|8845|9666|-8.49
|5	|BB Ball Bearing|8|2011|9666|12837|-24.7
|6	|BB Ball Bearing|7|2011|12837|5259|144.1
|7	|BB Ball Bearing|6|2011|5259|null|0.0
|8	|...

💡 Over the last 6 months of the year, the data shows fluctuations and variations in stock quantity for each product. There is a consistent decrease each quarter, suggesting that **the products were selling well and steadily**.

#### Query 7️⃣: Calc Ratio of Stock / Sales in 2011 by product name, by month *Order results by month desc, ratio desc. Round Ratio to 1 decimal mom yoy*
```sql
with sale_data as (
  select 
    extract(month from sod.ModifiedDate) as mth 
    ,extract(year from sod.ModifiedDate) as yr 
    ,sod.ProductID
    ,p.Name
    ,sum(sod.OrderQty) as sales
  from `adventureworks2019.Sales.SalesOrderDetail` as sod
  left join `adventureworks2019.Production.Product` as p 
    on sod.ProductID = p.ProductID
  where format_date("%Y", sod.ModifiedDate) = '2011'
  group by 1,2,3,4
),
stock_data as ( 
  select
    extract(month from ModifiedDate) as mth 
    ,extract(year from ModifiedDate) as yr 
    ,ProductID
    ,sum(StockedQty) as stocks
  from `adventureworks2019.Production.WorkOrder` 
  where format_date("%Y", ModifiedDate) = '2011'
  group by 1,2,3 
)
select 
  a.*
  ,b.stocks 
  ,round(coalesce(b.stocks,0)/a.sales,2) as ratio 
from sale_data as a 
left join stock_data as b 
  on a.ProductID = b.ProductID 
  and a.mth = b.mth 
  and a.yr = b.yr 
order by 1 desc, 7 desc ;
```
|Row	|mth|yr|ProductID|Name|sales|stocks|ratio
|-|-|-|-|-|-|-|-
|1	|12|2011|758|Road-450 Red, 52|37|518|14.0
|2	|12|2011|754|Road-450 Red, 58|29|348|12.0
|3	|12|2011|755|Road-450 Red, 60|18|162|9.0
|4	|12|2011|774|Mountain-100 Silver, 48|22|189|8.59
|5	|12|2011|762|Road-650 Red, 44|82|680|8.29
|6	|12|2011|756|Road-450 Red, 44|23|184|8.0
|7	|12|2011|761|Road-650 Red, 62|62|465|7.5
|8	|...

💡 Higher sales and a lower stock-to-sales ratio over the months indicate that the products were performing well in the market. This suggests strong demand and efficient inventory management.

#### Query 8️⃣: No of order and value at Pending status in 2014
```sql
select 
  extract(year from ModifiedDate) as yr
  ,Status
  ,count(PurchaseOrderID) as order_cnt
  ,sum(TotalDue) as value 
from `adventureworks2019.Purchasing.PurchaseOrderHeader` 
where format_timestamp('%Y', ModifiedDate) = '2014'
  and Status = 1
group by 1,2;
```
|Row	|yr|Status|order_cnt|value
|-|-|-|-|-
|1	|2014|1|224|3873579.0123000029

💡 The significant number of pending orders underscores the necessity of evaluating vendor performance to gauge their efficiency and recommending a revamp of their order processes.


---

## 🔎 Final Conclusion & Recommendations  

This SQL project, through its analysis of the AdventureWorks dataset, provided several valuable business insights:

- **Revenue and Quantity Sales**: Identifying Bike Racks with the highest sales revenue and Road Frames with the highest quantity sales informed inventory and marketing strategies to meet strong market demand. Notable year-over-year growth in Mountain Frames and Socks highlighted growth opportunities.
- **Monthly and Yearly Trends**: Tracking sales trends month-over-month and year-over-year revealed seasonal patterns and growth rates. This enabled businesses to optimize promotional activities, align inventory management, and allocate resources more efficiently.
- **Customer and Order Management**: Insights into customer retention emphasized the need for improved retention strategies. Observing stock quantity trends and the high number of pending orders underscored the importance of effective inventory and order process management to boost sales revenue.

Overall, this project demonstrated the power of data-driven strategies in optimizing business operations and making informed decisions, enhancing SQL skills, and showcasing the practical applications of data analysis in a business context.












