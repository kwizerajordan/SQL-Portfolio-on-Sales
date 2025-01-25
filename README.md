-- Portfolio: SQL Skills Demonstration
-- Project: Sales Analysis for CEKA250 (A Rwandan Shoe Company)

-- 1. Creating a Database for CEKA250
CREATE DATABASE CEKA250_Sales;
USE CEKA250_Sales;

-- 2. Creating Tables for the Database
-- Table: Products
CREATE TABLE Products (
    ProductID INT PRIMARY KEY AUTO_INCREMENT,
    ProductName VARCHAR(50) NOT NULL,
    Category VARCHAR(50),
    Price DECIMAL(10, 2),
    StockQuantity INT
);

-- Table: Customers
CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY AUTO_INCREMENT,
    City VARCHAR(50),
    Province VARCHAR(50),
    CustomerType VARCHAR(20), -- e.g., Retail or Wholesale
    Email VARCHAR(100),
    PhoneNumber VARCHAR(20)
);

-- Table: Sales
CREATE TABLE Sales (
    SaleID INT PRIMARY KEY AUTO_INCREMENT,
    SaleDate DATE,
    CustomerID INT,
    ProductID INT,
    Quantity INT,
    TotalAmount DECIMAL(10, 2),
    Currency VARCHAR(10) DEFAULT 'RWF',
    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID),
    FOREIGN KEY (ProductID) REFERENCES Products(ProductID)
);

-- 3. Inserting Sample Data
-- Products Table
INSERT INTO Products (ProductName, Category, Price, StockQuantity)
VALUES 
('Classic Sneakers', 'Shoes', 25000, 100),
('Running Shoes', 'Shoes', 45000, 80),
('Formal Leather Shoes', 'Shoes', 60000, 50),
('Sports Sandals', 'Sandals', 15000, 120),
('Winter Boots', 'Shoes', 80000, 30);

-- Customers Table
INSERT INTO Customers (City, Province, CustomerType, Email, PhoneNumber)
VALUES 
('Kigali', 'Kigali City', 'Retail', 'customer1@example.com', '0788001122'),
('Musanze', 'Northern Province', 'Wholesale', 'customer2@example.com', '0727002233'),
('Huye', 'Southern Province', 'Retail', 'customer3@example.com', '0788113344'),
('Rubavu', 'Western Province', 'Retail', 'customer4@example.com', '0788224455'),
('Kayonza', 'Eastern Province', 'Wholesale', 'customer5@example.com', '0788335566');

-- Sales Table
INSERT INTO Sales (SaleDate, CustomerID, ProductID, Quantity, TotalAmount)
VALUES 
('2025-01-01', 1, 1, 2, 50000),
('2025-01-02', 2, 2, 1, 45000),
('2025-01-03', 3, 3, 3, 180000),
('2025-01-04', 4, 4, 4, 60000),
('2025-01-05', 5, 5, 1, 80000);

-- 4. Basic SQL Queries
-- Total Sales Revenue
SELECT SUM(TotalAmount) AS TotalRevenue, Currency
FROM Sales
GROUP BY Currency;

-- Top 3 Best-Selling Products
SELECT p.ProductName, SUM(s.Quantity) AS TotalSold
FROM Sales s
JOIN Products p ON s.ProductID = p.ProductID
GROUP BY p.ProductName
ORDER BY TotalSold DESC
LIMIT 3;

-- Customer Purchase History
SELECT c.City, c.Province, c.CustomerType, p.ProductName, s.Quantity, s.TotalAmount, s.SaleDate
FROM Sales s
JOIN Customers c ON s.CustomerID = c.CustomerID
JOIN Products p ON s.ProductID = p.ProductID
ORDER BY s.SaleDate;

-- 5. Advanced SQL Queries
-- Monthly Revenue Report
SELECT DATE_FORMAT(SaleDate, '%Y-%m') AS Month, SUM(TotalAmount) AS MonthlyRevenue, Currency
FROM Sales
GROUP BY DATE_FORMAT(SaleDate, '%Y-%m'), Currency;

-- Stock Alert: Products with Low Stock
SELECT ProductName, StockQuantity
FROM Products
WHERE StockQuantity < 20;

-- Sales by Province
SELECT c.Province, SUM(s.TotalAmount) AS TotalSales
FROM Sales s
JOIN Customers c ON s.CustomerID = c.CustomerID
GROUP BY c.Province
ORDER BY TotalSales DESC;

-- Predicting Stock Depletion
WITH RollingSales AS (
    SELECT 
        ProductID, 
        AVG(Quantity) OVER (PARTITION BY ProductID ORDER BY SaleDate ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS AvgSales
    FROM Sales
)
SELECT p.ProductName, p.StockQuantity, rs.AvgSales, 
       CASE 
           WHEN rs.AvgSales > 0 THEN ROUND(p.StockQuantity / rs.AvgSales, 2)
           ELSE NULL
       END AS DaysUntilStockDepletion
FROM Products p
LEFT JOIN RollingSales rs ON p.ProductID = rs.ProductID;
