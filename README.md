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

## II. Eploring the Dataset

### Query 01: Calc Quantity of items, Sales value & Order quantity by each Subcategory in Last 12 Mth
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

### Query 02: Calc % YoY growth rate by SubCategory & release top 3 cat with highest grow rate. Can use metric: quantity_item. Round results to 2 decimal
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

"Road Frames" was the product with the highest quantity sales, while "Mountain Frames" and "Socks" have the growth rate YoY%

### Query 03: Query 3: Ranking Top 3 TeritoryID with biggest Order quantity of every year. If there's TerritoryID with same quantity in a year, do not skip the rank number
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



### Query 04: Query 4: Calc Total Discount Cost belongs to Seasonal Discount for each SubCategory
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

There was a substantial increase in the discount cost from 2012 to 2013 for the "Helmets" subcategory, indicating a more aggressive discount strategy or higher sales volume benefiting from seasonal discounts.

### Query 05: Query 5: Retention rate of Customer in 2014 with status of Successfully Shipped (Cohort Analysis)
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

There is a notable drop in retention from the first month to subsequent months. Most customers who made their first purchase in a given month did not return in significant numbers in the following months.
This pattern is consistent across all months analyzed, indicating a need for strategies to improve customer retention.

### Query 06: Trend of Stock level & MoM diff % by all product in 2011. If %gr rate is null then 0. Round to 1 decimal
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




### Query 07: Calc Ratio of Stock / Sales in 2011 by product name, by month *Order results by month desc, ratio desc. Round Ratio to 1 decimal mom yoy*
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




### Query 08: No of order and value at Pending status in 2014
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


## III. Conclusion
This SQL project delved into the AdventureWorks dataset, revealing key insights into sales trends, customer demographics, and product performance. The analysis highlights the power of data-driven strategies in optimizing business operations and making informed decisions. Through this project, I enhanced my SQL skills and demonstrated the practical applications of data analysis in a business context.












