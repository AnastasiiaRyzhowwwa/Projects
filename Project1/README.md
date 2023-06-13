# Анализ интернет продаж
Цель анализа улучшить отчёты о продажах в интернете, путем перехода от статического отчёта к визуальному мониторингу. Надо обратить внимание на:
1. продукты, 
2. количество,
3. клиентов, 
4. время покупок

Важно учесть, что каждый продавец работает со своим продуктом и клиентом.
Сравнить ценность производства с бюджетом предприятия.
Данные берутся за последние 3 года.
Необходимая система: Power BI

| 1  | Менеджер по продажам | Создание дашборда обзор на интернет продажи | Следить какие клиенты и какие продукты покупаются лучше | Дашборд обновляет данные 1 раз в день |
| ------------- | ------------- | ------------- | ------------- | ------------- |
| 2  | Продавец отдела  | Детальный обзор интернет продаж покупателям | Следить за клиентами которые покупают больше всего и кому мы можем продать больше | Дашборд фильтрует данные для каждого клиента |
| 3  | Продавец отдела  | Детальный обзор интернет продаж по покупкам | Следить за продуктами которые покупаются больше | Дашборд фильтрует данные для каждого продукта |
| 4  | Менеджер по продажам  | Создание дашборда обзор на интернет продажи | Следить за продажами с течением времени в соответствии с бюджетом | Дашборд сравнение KPI с бюджетом |

***
# Очистка и преобразование данных
## 1.	Очистим таблицу dbo.DimDate, где взяты значения с 2019 года и выше
```sql
SELECT 
  [DateKey], 
  [FullDateAlternateKey] AS Date, 
  [EnglishDayNameOfWeek] AS Day, 
  [WeekNumberOfYear] AS WeekNr, 
  [EnglishMonthName] AS Month, 
  LEFT([EnglishMonthName], 3) AS MonthShort #Берет первые 3 буквы, 
  [MonthNumberOfYear] AS MonthNo, 
  [CalendarQuarter] AS Quarter, 
  [CalendarYear] AS Year 
FROM 
  [AdventureWorksDW2019].[dbo].[DimDate] 
WHERE 
  CalendarYear >= 2019
```
## 2. Возьмём таблицу покупателей и соединим её по ключу geographykey с таблицей DimGeography по левому краю, полученное значение сортируется в порядке возрастания по ключу покупателя
```sql
SELECT 
  c.customerkey AS CustomerKey, 
  --,[GeographyKey]
  --,[CustomerAlternateKey]
  --,[Title]
  c.firstname AS [First Name], 
  --,[MiddleName]
  c.lastName AS [Last Name], 
  c.firstname + ' ' + c.lastName AS [Full Name], #добавит столбец соединяющий и. и фам
  --,[NameStyle]
  --,[BirthDate]
  --,[MaritalStatus]
  --,[Suffix]
  CASE c.gender WHEN 'M' THEN 'Male' WHEN 'F' THEN 'Female' END AS [Gender],#переменует в Male and Female
  --,[EmailAddress]
  --,[YearlyIncome]
  --,[TotalChildren]
  --,[NumberChildrenAtHome]
  --,[EnglishEducation]
  --,[SpanishEducation]
  --,[FrenchEducation]
  --,[EnglishOccupation]
  --,[SpanishOccupation]
  --,[FrenchOccupation]
  --,[HouseOwnerFlag]
  --,[NumberCarsOwned]
  --,[AddressLine1]
  --,[AddressLine2]
  --,[Phone]
  c.datefirstpurchase AS [DateFirstPurchase], 
  --,[CommuteDistance]
  g.city AS [Customer City] 
FROM 
  [AdventureWorksDW2019].[dbo].[DimCustomer] AS c 
  LEFT JOIN [AdventureWorksDW2019].[dbo].[DimGeography] AS g ON g.geographykey = c.geographykey 
ORDER BY 
  CustomerKey ASC
```
## 3. Получаем таблицу о товаре объединённую с 2 двумя другими, чтобы получить из них подкатегорию и категорию
```sql
SELECT 
  p.[ProductKey], 
  p.[ProductAlternateKey] AS ProductItemCode,
  p.[EnglishProductName] AS [Product Name],
  ps.EnglishProductSubcategoryName AS [Sub Category], 
  pc.EnglishProductCategoryName AS [Product Category], 
  p.[Color] AS [Product Color], 
  p.[Size] AS [Product Size], 
  p.[ProductLine] AS [Product Model Name], 
  p.[ModelName] AS [Product Model Name], 
  p.[EnglishDescription] AS [Product Description], 
  ISNULL(p.[Status], 'Outdated') AS [Product Status] #если значение null то выводим 'Outdated'
FROM 
  [AdventureWorksDW2019].[dbo].[DimProduct] AS p 
  LEFT JOIN [AdventureWorksDW2019].[dbo].[DimProductSubcategory] AS ps ON ps.ProductSubcategoryKey = p.ProductSubcategoryKey 
  LEFT JOIN [AdventureWorksDW2019].[dbo].[DimProductCategory] AS pc ON ps.ProductCategoryKey = pc.ProductCategoryKey 
ORDER BY 
  p.ProductKey ASC
```
## 4. Получаем таблицу интернет продаж за последние 3 года
```sql
SELECT  [ProductKey]
      ,[OrderDateKey]
      ,[DueDateKey]
      ,[ShipDateKey]
      ,[CustomerKey]
      ,[SalesOrderNumber]
      ,[SalesAmount]
  FROM [AdventureWorksDW2019].[dbo].[FactInternetSales]
  WHERE LEFT (OrderDateKey, 4)>=YEAR(GETDATE())-3 # Берем с лева 4 значения у OrderDateKey(получаем год) и сравниваем с текущей датой вычитая последние 3 года
  ORDER BY OrderDateKey ASC
```
