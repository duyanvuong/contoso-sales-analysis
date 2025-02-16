# Project Introduction
This project leverages the ContosoRetailDW database by Microsoft, focusing on the FactSales table to analyze retail sales data. The objective is to uncover actionable insights by exploring sales performance, customer purchasing behavior, and trends over time. By integrating key dimensions such as products, geography, and dates, the analysis provides a comprehensive understanding of business performance. The outcomes of this project include interactive visualizations, data-driven dashboards, and valuable metrics to support strategic decision-making in retail operations.

# Table of Content
## [Database setup locally and EDA using SQL Server](/sql%20queries/Database_Setup_EDA_README.md)

- Database set up
- Data cleaning, updating

## [Data Inspectation Using Python](/python/01_Data_Inspectation.ipynb)

- Inspecting data to get a glimpse of the dataset

## [EDA Using SQL](/sql%20queries/02_Analytics/)

- EDA using SQL Server
    - Sales Performance Analysis
    - Profitability Analysis
    - Customer Behavior
    - Regional Performance
    - Time - based Analysis

## [Visualizing data using Python](/python/)

- Core libraries: `Pandas`, `Matplotlib`, `Seaborn`
- Conect to the database using `SQLAlchemy`
- Visualizing data from views created from SQL Server

# Main Content

## 1. Database Setup and Preparation using SQL

### 1.1. Create Database

#### [Source code here](/sql%20queries/01_Database_Setup/01_Create_Database.sql)

The database that I have created will include
- 1 Fact table
    - FactSales
- 8 Dimensions
    - DimDate
    - DimChannel
    - DimPromotion
    - DimStore, DimGeography
    - DimProduct, DimProductSubcategory, DimProductCategory

![data model](/data%20model/contoso%20data%20model.jpeg)

### 1.2. Extracting data

#### [Source code here](/sql%20queries/01_Database_Setup/02_Extracting_Data.sql)

All the data are extracted from ContosoRetailDW using the following SQL scripts
```sql
insert into DimDate (DateKey, Year, MonthName, Day, Quarter)
select
	DateKey,
	CalendarYear as Year,
	CalendarMonthLabel as MonthName,
	DAY(DateKey) as Day,
	CalendarQuarterLabel as Quarter
from ContosoRetailDW..DimDate
```
Different columns will be selected for different dimensions

### 1.3. Data cleaning

#### [Source code here](/sql%20queries/01_Database_Setup/04_Data_Cleaning.sql)

```sql
select 
    COUNT(*) as TotalRows,
    SUM(IIF(ContinentName is null, 1, 0)) as MissingValueContinentName,
    SUM(IIF(CityName is null, 1, 0)) as MissingValuesCityName,
    SUM(IIF(StateProvinceName is null, 1, 0)) as MissingValuesStateProvinceName,
    SUM(IIF(RegionCountryName is null, 1, 0)) as MissingValuesRegionCountryName
from DimGeography
```

After observing each table using the previous SQL script, I realized that only in `DimGeography` has null values. Those null values is stand for `Continent` (`CityName`, `StateProvince` and `RegionCountryName` are null).
Because of that, I decided to remove all null values that exist in `DimGeography using the following SQL scripts

```sql
delete from DimGeography
where CityName IS NULL
```

### 1.4. Updating data

#### [Source code here](/sql%20queries/01_Database_Setup/04_Data_Cleaning.sql)

In the origin data warehouse provided by Microsoft, I realized that the date column is all the way to 2007 and I want the data is up to date. So I decided to update `DateKey` column to the most recent time.

```sql
update DimDate
set DateKey = CASE 
					WHEN YEAR(DateKey) = 2005 THEN DATEADD(YEAR, 2018 - 2005, DateKey)
					WHEN YEAR(DateKey) = 2006 THEN DATEADD(YEAR, 2019 - 2006, DateKey)
					WHEN YEAR(DateKey) = 2007 THEN DATEADD(YEAR, 2020 - 2007, DateKey)
					WHEN YEAR(DateKey) = 2008 THEN DATEADD(YEAR, 2021 - 2008, DateKey)
					WHEN YEAR(DateKey) = 2009 THEN DATEADD(YEAR, 2022 - 2009, DateKey)
					WHEN YEAR(DateKey) = 2010 THEN DATEADD(YEAR, 2023 - 2010, DateKey)
					WHEN YEAR(DateKey) = 2011 THEN DATEADD(YEAR, 2024 - 2011, DateKey)
				END
```