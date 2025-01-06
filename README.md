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
where date(sod.ModifiedDate) >= (select date_add(max(date(sod.ModifiedDate)), interval -12 month)
                             from `adventureworks2019.Sales.SalesOrderDetail`)
group by 1,2
order by 2,1 desc;
```

### Query 02: Calc % YoY growth rate by SubCategory & release top 3 cat with highest grow rate. Can use metric: quantity_item. Round results to 2 decimal
```sql
SELECT distinct
  trafficSource.source
  , count(totals.visits) total_visits
  , count(totals.bounces) total_no_of_bounces
  , round(count(totals.bounces)*100.0 /count(totals.visits), 3) bounce_rates
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
group by 1
order by 2 desc, 3 desc;
```

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
  ,sum(TotalDue) as value -- em thắc mắc tại sao ở đây mình dùng TotalDue mà không phải SubTotal ạ
                            --> vì em nghĩ đơn bị pending là đơn có vấn đề không thanh toán được thì sao lại tính thuế vào ạ?
from `adventureworks2019.Purchasing.PurchaseOrderHeader` 
where format_timestamp('%Y', ModifiedDate) = '2014'
  and Status = 1
group by 1,2;
```














