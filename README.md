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

### Query 01: Calculate the Quantity of items, Sales value & Order quantity by each Subcategory in Last 12 Mth
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

### Query 02: 
```sql

```

### Query 03: 
```sql

```

### Query 04: 
```sql

```

### Query 05: 
```sql

```

### Query 06: 
```sql

```

### Query 07: 
```sql

```

### Query 08: 
```sql

```














