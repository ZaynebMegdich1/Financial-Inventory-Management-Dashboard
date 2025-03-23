# **DAX Measures for Financial & Inventory Management Dashboard**  

## **1. Financial Performance Measures**  

### **1.1 Total Revenue**  
```DAX
Total Revenue = SUM(Stock[Revenue])
```  
This measure calculates the total revenue generated from sales.  

### **1.2 Revenue Previous Month (PM)**  
```DAX
Revenue PM = CALCULATE(SUM(Stock[Revenue]), PREVIOUSMONTH('Date'[Date]))
```  
This measure calculates the total revenue for the previous month.  

### **1.3 Sales Variance**  
```DAX
Sales Variance = [Total Revenue] - [Revenue PM]
```  
This measure calculates the difference in revenue compared to the previous month.  

### **1.4 Revenue Variance Percentage**  
```DAX
Revenue Var % = DIVIDE([Sales Variance], [Total Revenue], 0)
```  
This measure calculates the percentage change in revenue compared to the previous month.  

---

## **2. Order Analysis Measures**  

### **2.1 Number of Orders**  
```DAX
NumberOfOrders = COUNTROWS('Past Orders')
```  
This measure calculates the total number of orders placed.  

### **2.2 Orders Previous Month (PM)**  
```DAX
Orders PM = CALCULATE([NumberOfOrders], PREVIOUSMONTH('Date'[Date]))
```  
This measure calculates the number of orders in the previous month.  

### **2.3 Orders Variance**  
```DAX
Orders Variance = [NumberOfOrders] - [Orders PM]
```  
This measure calculates the difference in the number of orders compared to the previous month.  

### **2.4 Orders Variance Percentage**  
```DAX
Orders Var % = DIVIDE([Orders Variance], [NumberOfOrders], 0)
```  
This measure calculates the percentage change in the number of orders compared to the previous month.  

---

## **3. Inventory Management Measures**  

### **3.1 Stock Overview**  

#### **Total Inventory Value**  
```DAX
TotalInventoryValue = SUM(Stock[Total Stock Value])
```  
This measure calculates the total value of the inventory in stock.  

#### **Stock Quantity**  
```DAX
Stock Quantity = SUM(Stock[Current Stock Quantity])
```  
This measure calculates the total quantity of stock available.  

#### **Average Stock Quantity**  
```DAX
AverageStock = AVERAGE(Stock[Current Stock Quantity])
```  
This measure calculates the average stock quantity available.  

#### **Number of Items**  
```DAX
Number of Items = DISTINCTCOUNT(Stock[SKU ID])
```  
This measure calculates the total number of unique items in stock.  

---

### **3.2 Stock Movement & Turnover**  

#### **Total Order Quantity**  
```DAX
TotalOrderQuantity = SUM('Past Orders'[Order Quantity])
```  
This measure calculates the total quantity of orders placed.  

#### **Quantity Sold**  
```DAX
Quantity Sold = SUM(Stock[Sale Quantity])
```  
This measure calculates the total quantity of products sold.  

#### **Quantity Sold Previous Month (PM)**  
```DAX
Quantity Sold PM = CALCULATE([Quantity Sold], PREVIOUSMONTH('Date'[Date]))
```  
This measure calculates the quantity of products sold in the previous month.  

#### **Quantity Sold Variance**  
```DAX
Quantity Sold Variance = [Quantity Sold] - [Quantity Sold PM]
```  
This measure calculates the difference in quantity sold compared to the previous month.  

#### **Quantity Sold Variance Percentage**  
```DAX
Quantity Var % = DIVIDE([Quantity Sold Variance], [Quantity Sold], 0)
```  
This measure calculates the percentage change in quantity sold compared to the previous month.  

#### **Inventory Turnover**  
```DAX
InventoryTurnover = DIVIDE([TotalOrderQuantity], [AverageStock], 0)
```  
This measure calculates the inventory turnover rate, indicating how many times stock is sold and replaced over a period.  

---

### **3.3 Stock Levels & Reordering**  

#### **Stock Out Rate**  
```DAX
StockOut Rate = 
VAR noStock = CALCULATE(COUNTROWS(Stock), Stock[Current Stock Quantity] = 0)
VAR NumberOfStock = COUNTROWS(Stock)
RETURN IF(
    ISBLANK(DIVIDE(noStock, NumberOfStock, 0)), 
    0, 
    DIVIDE(noStock, NumberOfStock, 0)
)
```  
This measure calculates the stock-out rate by determining the percentage of out-of-stock items.  

#### **Out of Stock Products**  
```DAX
Out of Stock Products = 
VAR OutOfStockTable = FILTER(Stock, Stock[Stock Status] = "Out Of Stock")
RETURN CONCATENATEX(OutOfStockTable, Stock[SKU ID], ", ")
```  
This measure lists all SKUs that are currently out of stock.  

#### **Out of Stock Count**  
```DAX
OutStock Number = 
CALCULATE(COUNTROWS(Stock), FILTER(Stock, Stock[Stock Status] = "Out Of Stock"))
```  
This measure calculates the number of out-of-stock products.  

#### **Average Daily Sales**  
```DAX
Avg Daily Sales = 
VAR TotalSales = SUM('Past Orders'[Order Quantity])
VAR TotalDays = DISTINCTCOUNT('Date'[Date])
RETURN DIVIDE(TotalSales, TotalDays, 0)
```  
This measure calculates the average number of units sold per day.  

#### **Maximum Daily Sales**  
```DAX
Max Daily Sales = 
VAR DailySales = 
    SUMMARIZE(
        'Past Orders',
        'Date'[Date],
        'Past Orders'[SKU ID],
        "Daily Sales", SUM('Past Orders'[Order Quantity])
    )
RETURN 
    MAXX(DailySales, [Daily Sales])
```  
This measure calculates the highest number of units sold in a single day.  

#### **Safety Stock**  
```DAX
Safety Stock = 
VAR MaxDailySales = [Max Daily Sales]
VAR AvgDailySales = [Avg Daily Sales]
VAR MaxLeadTime = MAX('Stock'[Maximum Lead Time (days)])
VAR AvgLeadTime = AVERAGE('Stock'[Average Lead Time (days)])
RETURN 
    (MaxDailySales * MaxLeadTime) - (AvgDailySales * AvgLeadTime)
```  
This measure calculates the safety stock, ensuring that stock is available to handle unexpected demand.  

#### **Reorder Point**  
```DAX
Reorder Point = 
VAR AvgDailySales = [Avg Daily Sales]
VAR AvgLeadTime = AVERAGE('Stock'[Average Lead Time (days)])
VAR SafetyStock = [Safety Stock]
RETURN 
    (AvgDailySales * AvgLeadTime) + SafetyStock
```  
This measure calculates the reorder point, which determines when new stock should be ordered.  

#### **Reorder Point Alert**  
```DAX
Reorder Point Alert = 
VAR reorder = CALCULATE(
    COUNT('Stock'[SKU ID]),
    FILTER('Stock', 'Stock'[Current Stock Quantity] <= [Reorder Point])
)
VAR out = CALCULATE(COUNTROWS(Stock), Stock[Stock Status] = "Out Of Stock")
RETURN reorder - out
```  
This measure calculates the number of products that have reached or passed their reorder point.  

#### **Restock Indicator**  
```DAX
Need Restock = 
IF( 
     [Stock Quantity] <= [Reorder Point], 
    "Restock", 
    "Not Yet" 
)
```  
This measure provides a simple "Restock" or "Not Yet" status for products based on their reorder point.  

---

### **3.4 Time-Based Insights**  

#### **Previous Month Name**  
```DAX
PreviousMonthName = 
FORMAT(EOMONTH(MAX('Date'[Date]), -1), "MMMM") & ": "
```  
This measure dynamically retrieves the name of the previous month for reporting purposes.  

---
