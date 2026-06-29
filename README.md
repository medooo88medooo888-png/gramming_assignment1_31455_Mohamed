# Individual Assignment I: CTEs & SQL Window Functions Project
**Course:** Database Programming (DPR400210)  
**Instructor:** Eric Maniraguha  
**Institution:** University of Lay Adventists of Kigali (UNILAK)  

---
## Student Information
* **Full Name:** Mohamed
* **Student ID:** 31455
---

## 1. Business Scenario
This project implements an **E-Commerce Sales Management System** designed to track customers, products, and individual sales orders. The database schema allows the sales management team to extract strategic business insights using advanced SQL querying techniques, specifically Common Table Expressions (CTEs) and SQL Window Functions.

---

## 2. Database Schema
The system consists of three related tables:
* **Customers:** Stores unique buyer information (`CustomerID`, `CustomerName`, `Country`).
* **Products:** Contains product catalogs, pricing, and groupings (`ProductID`, `ProductName`, `Category`, `Price`).
* **Orders:** Connects customers and products while managing transaction details, quantities, dates, and organizational hierarchies (`OrderID`, `CustomerID`, `ProductID`, `OrderDate`, `Quantity`, `TotalAmount`, `EmployeeID`, `ManagerID`).

---

## 3. Part A: Common Table Expressions (CTEs)

### 1. Simple CTE
```sql
WITH HighValueOrders AS (
    SELECT OrderID, CustomerID, TotalAmount 
    FROM Orders
    WHERE TotalAmount > 800
)
SELECT * FROM HighValueOrders;
### 2. Multiple CTEs
```sql
WITH RwandaCustomers AS (
    SELECT CustomerID, CustomerName FROM Customers WHERE Country = 'Rwanda'
),
RecentOrders AS (
    SELECT OrderID, CustomerID, TotalAmount FROM Orders WHERE OrderDate >= '2026-01-01'
)
SELECT rc.CustomerName, ro.OrderID, ro.TotalAmount
FROM RwandaCustomers rc
JOIN RecentOrders ro ON rc.CustomerID = ro.CustomerID;
WITH RECURSIVE SalesHierarchy AS (
    SELECT EmployeeID, ManagerID, 1 AS ManagementLevel
    FROM Orders WHERE ManagerID IS NULL
    UNION ALL
    SELECT o.EmployeeID, o.ManagerID, sh.ManagementLevel + 1
    FROM Orders o
    JOIN SalesHierarchy sh ON o.ManagerID = sh.EmployeeID
)
SELECT DISTINCT * FROM SalesHierarchy;
WITH CustomerStats AS (
    SELECT CustomerID, COUNT(OrderID) AS TotalOrders, SUM(TotalAmount) AS TotalSpent
    FROM Orders
    GROUP BY CustomerID
)
SELECT cs.*, c.CustomerName 
FROM CustomerStats cs
JOIN Customers c ON cs.CustomerID = c.CustomerID;
WITH SalesByProduct AS (
    SELECT ProductID, SUM(Quantity) AS TotalQty
    FROM Orders
    GROUP BY ProductID
)
SELECT p.ProductName, p.Category, s.TotalQty, (s.TotalQty * p.Price) AS Revenue
FROM SalesByProduct s
JOIN Products p ON s.ProductID = p.ProductID;
SELECT 
    o.OrderID, p.Category, o.TotalAmount,
    ROW_NUMBER() OVER(PARTITION BY p.Category ORDER BY o.TotalAmount DESC) AS RowNum,
    RANK() OVER(PARTITION BY p.Category ORDER BY o.TotalAmount DESC) AS RankNum,
    DENSE_RANK() OVER(PARTITION BY p.Category ORDER BY o.TotalAmount DESC) AS DenseRankNum,
    NTILE(2) OVER(ORDER BY o.TotalAmount DESC) AS SalesTile
FROM Orders o
JOIN Products p ON o.ProductID = p.ProductID;
SELECT 
    OrderID, OrderDate, TotalAmount,
    SUM(TotalAmount) OVER(ORDER BY OrderDate) AS CumulativeSales,
    AVG(TotalAmount) OVER() AS OverallAverage,
    LAG(TotalAmount, 1) OVER(ORDER BY OrderDate) AS PreviousOrderAmount,
    LEAD(TotalAmount, 1) OVER(ORDER BY OrderDate) AS NextOrderAmount
FROM Orders;
