![1711408967531](https://github.com/user-attachments/assets/58453876-951e-4fdd-9cb4-907d6e9dae7a)

# 📊 Project Title: AdventureWorks Dataset Exploring  
Author: Tri Nguyen  
Date: Nov. 10, 2024 
Tools Used: SQL - BigQuery Platform 

---

## 📑 Table of Contents  
1. [📌 Background & Overview](#-background--overview)  
2. [📂 Dataset Description & Data Structure](#-dataset-description--data-structure)  
3. [🧠 Design Thinking Process](#-design-thinking-process)  
4. [📊 Key Insights & Visualizations](#-key-insights--visualizations)  
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
- **Stock and Order Management**: Stock levels consistently decreased each quarter, suggesting strong sales. High pending order counts highlight the necessity of revamping the order process to drive higher sales revenue.

---

## 📂 Dataset Description & Data Structure  

### 📌 Data Source  
- Source: The AdventureWorks2019 dataset by Microsoft is a comprehensive sample database that simulates a manufacturing company's operations.
<details>
<summary>Click to toggle contents of `code`</summary>

```
CODE!
```
</details>

- Size: (Mention the number of rows & columns)  
- Format: (.csv, .sql, .xlsx, etc.)  

### 📊 Data Structure & Relationships  

#### 1️⃣ Tables Used:  
Mention how many tables are in the dataset.  

#### 2️⃣ Table Schema & Data Snapshot  

Table 1: Products Table  

👉🏻 Insert a screenshot of table schema 

 _Example:_

| Column Name | Data Type | Description |  
|-------------|----------|-------------|  
| Product_ID  | INT      | Unique identifier for each product |  
| Name        | TEXT     | Product name |  
| Category    | TEXT     | Product category |  
| Price       | FLOAT    | Price per unit |  



Table 2: Sales Transactions  

👉🏻 Insert a screenshot of table schema 


 _Example:_

| Column Name    | Data Type | Description |  
|---------------|----------|-------------|  
| Transaction_ID | INT      | Unique identifier for each sale |  
| Product_ID     | INT      | Foreign key linking to Products table |  
| Quantity       | INT      | Number of items sold |  
| Sale_Date      | DATE     | Date of transaction |  



#### 3️⃣ Data Relationships:  
Describe the connections between tables—e.g., one-to-many, many-to-many.  

👉🏻 Include a screenshot of Data Modeling to visualize relationships.  

---




























# [SQL] AdventureWorks Dataset Exploring 
## I. Introduction 
### Overview
This SQL project leverages advanced techniques to analyze the AdventureWorks dataset, a comprehensive example of relational database management. By employing complex queries, data mining, and performance optimization strategies, this initiative aims to uncover insightful trends, enhance data visualization, and support informed decision-making within a simulated business environment.

### Dataset Access 
The AdventureWorks2019 dataset by Microsoft is a comprehensive sample database simulating a manufacturing company's operations, ideal for advanced SQL queries, data analysis, and database management practice. To access the dataset, follow these steps:

- Log in to your Google Cloud Platform account and create a new project.
- Navigate to the BigQuery console and select your newly created project.
- In the navigation panel, select "Add Data" and then choose "Star a project by name".
- Enter the project name **"adventureworks2019"** and click "Enter".
- Click on the **"adventureworks2019"** table to open it.

## II. Insight

- Product's sale revenue, sale quantity were performing well in the market. 
- Revamp the order process to reduce pending orders, which in turn can drive higher sales revenue.

## III. Eploring the Dataset
### Apply Problem Solving 

| Understand Problem | Break it down into smaller pieces | Ideate | Implement and Review 
|-|-|-|-
| Which to be calculated (sum, count, ratio, etc.) and grouped? <br> Does it needs any filter (time, conditions, etc.)? | Which tables have data that I want to get? <br> Which columns have data corresponded to the problem? <br> Can I get the data I by one step, if not, break down even smaller? | How many step did I need to get the final result? <br> Is it optimized? | 

### Implement Step 
#### Query 01: Calc Quantity of items, Sales value & Order quantity by each Subcategory in Last 12 Mth
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

Bike Racks have had the highest sales revenue in the past 12 months, indicating strong market demand.

#### Query 02: Calc % YoY growth rate by SubCategory & release top 3 cat with highest grow rate. Can use metric: quantity_item. Round results to 2 decimal
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

Road Frames had the highest quantity sales, while Mountain Frames and Socks showed the highest _year-over-year_ growth rates.

#### Query 03: Query 3: Ranking Top 3 TeritoryID with biggest Order quantity of every year. If there's TerritoryID with same quantity in a year, do not skip the rank number
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

Territory No. 4 consistently had the highest order rates each year, with Territory No. 6 following closely behind.

#### Query 04: Query 4: Calc Total Discount Cost belongs to Seasonal Discount for each SubCategory
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

The "Helmets" sub-category was the only one with discount promotions in both 2012 and 2013. <br> 
There was a significant increase in discount costs from 2012 to 2013, suggesting a more aggressive discount strategy or higher sales volumes benefiting from seasonal discounts.

#### Query 05: Query 5: Retention rate of Customer in 2014 with status of Successfully Shipped (Cohort Analysis)
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

Retention drops significantly from the first month to subsequent months, with few customers returning after their initial purchase. This trend is consistent across all months analyzed, highlighting **the need for improved customer retention strategies**.

#### Query 06: Trend of Stock level & MoM diff % by all product in 2011. If %gr rate is null then 0. Round to 1 decimal
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

Over the last 6 months of the year, the data shows fluctuations and variations in stock quantity for each product. There is a consistent decrease each quarter, suggesting that **the products were selling well and steadily**.

#### Query 07: Calc Ratio of Stock / Sales in 2011 by product name, by month *Order results by month desc, ratio desc. Round Ratio to 1 decimal mom yoy*
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

Higher sales and a lower stock-to-sales ratio over the months indicate that the products were performing well in the market. This suggests strong demand and efficient inventory management.

#### Query 08: No of order and value at Pending status in 2014
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

The high number of pending orders highlights the need to **revamp the order process**. Doing so can help reduce pending orders and drive higher sales revenue.

## IV. Conclusion 
This SQL project delved into the AdventureWorks dataset, revealing key insights into sales trends, customer demographics, and product performance. The analysis highlights the power of data-driven strategies in optimizing business operations and making informed decisions. Through this project, I enhanced my SQL skills and demonstrated the practical applications of data analysis in a business context.








