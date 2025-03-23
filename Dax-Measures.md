# **DAX Measures & Calculated Columns for Financial and Inventory Management Dashboard**  

## **1. Financial Dashboard Measures**  

### **1.1 Total Revenue**  
```DAX
Total Revenue = SUM(Stock[Revenue])
```  
This measure calculates the total revenue generated from stock sales.  

### **1.2 Revenue for Previous Month**  
```DAX
Revenue Previous Month = CALCULATE([Total Revenue], PREVIOUSMONTH('Date'[Date]))
```  
This measure calculates the total revenue for the previous month.  

### **1.3 Revenue Variance**  
```DAX
Revenue Variance = [Total Revenue] - [Revenue Previous Month]
```  
This measure calculates the absolute difference in revenue compared to the previous month.  

### **1.4 Revenue Variance Percentage**  
```DAX
Revenue Variance % = DIVIDE([Revenue Variance], [Total Revenue], 0)
```  
This measure calculates the percentage change in revenue compared to the previous month.  

### **1.5 Total Orders**  
```DAX
Total Orders = COUNTROWS('Past Orders')
```  
This measure calculates the total number of customer orders.  

### **1.6 Orders for Previous Month**  
```DAX
Orders Previous Month = CALCULATE([Total Orders], PREVIOUSMONTH('Date'[Date]))
```  
This measure calculates the total number of orders placed in the previous month.  

### **1.7 Orders Variance**  
```DAX
Orders Variance = [Total Orders] - [Orders Previous Month]
```  
This measure calculates the absolute change in the number of orders compared to the previous month.  

### **1.8 Orders Variance Percentage**  
```DAX
Orders Variance % = DIVIDE([Orders Variance], [Total Orders], 0)
```  
This measure calculates the percentage change in the number of orders compared to the previous month.  

### **1.9 Total Quantity Sold**  
```DAX
Total Quantity Sold = SUM(Stock[Sale Quantity])
```  
This measure calculates the total quantity of items sold.  

### **1.10 Quantity Sold for Previous Month**  
```DAX
Quantity Sold Previous Month = CALCULATE([Total Quantity Sold], PREVIOUSMONTH('Date'[Date]))
```  
This measure calculates the quantity of items sold in the previous month.  

### **1.11 Quantity Sold Variance**  
```DAX
Quantity Sold Variance = [Total Quantity Sold] - [Quantity Sold Previous Month]
```  
This measure calculates the absolute difference in quantity sold compared to the previous month.  

### **1.12 Quantity Sold Variance Percentage**  
```DAX
Quantity Sold Variance % = DIVIDE([Quantity Sold Variance], [Total Quantity Sold], 0)
```  
This measure calculates the percentage change in quantity sold compared to the previous month.  

---

## **2. Inventory Management Measures**  

### **2.1 Total Inventory Value**  
```DAX
Total Inventory Value = SUM(Stock[Total Stock Value])
```  
This measure calculates the total value of the current inventory.  

### **2.2 Total Stock Quantity**  
```DAX
Total Stock Quantity = SUM(Stock[Current Stock Quantity])
```  
This measure calculates the total quantity of items available in stock.  

### **2.3 Average Stock Quantity**  
```DAX
Average Stock Quantity = AVERAGE(Stock[Current Stock Quantity])
```  
This measure calculates the average stock quantity across all items.  

### **2.4 Total Order Quantity**  
```DAX
Total Order Quantity = SUM('Past Orders'[Order Quantity])
```  
This measure calculates the total number of items ordered.  

### **2.5 Average Daily Sales**  
```DAX
Average Daily Sales = 
VAR TotalSales = [Total Order Quantity]
VAR TotalDays = DISTINCTCOUNT('Date'[Date])
RETURN DIVIDE(TotalSales, TotalDays, 0)
```  
This measure calculates the average number of items sold per day.  

### **2.6 Maximum Daily Sales**  
```DAX
Maximum Daily Sales = 
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
This measure calculates the highest number of items sold on a single day.  

### **2.7 Inventory Turnover Ratio**  
```DAX
Inventory Turnover Ratio = DIVIDE([Total Order Quantity], [Average Stock Quantity], 0)
```  
This measure calculates how frequently inventory is sold and replaced.  

### **2.8 Stockout Rate**  
```DAX
Stockout Rate = 
VAR NoStock = CALCULATE(COUNTROWS(Stock), Stock[Current Stock Quantity] = 0)
VAR TotalStockItems = COUNTROWS(Stock)
RETURN DIVIDE(NoStock, TotalStockItems, 0)
```  
This measure calculates the percentage of out-of-stock items.  

### **2.9 Out-of-Stock Items Count**  
```DAX
Out of Stock Count = 
CALCULATE(COUNTROWS(Stock), FILTER(Stock, Stock[Stock Status] = "Out Of Stock"))
```  
This measure counts the number of items currently out of stock.  

### **2.10 Out-of-Stock Products List**  
```DAX
Out of Stock Products = 
VAR OutOfStockTable = FILTER(Stock, Stock[Stock Status] = "Out Of Stock")
RETURN 
CONCATENATEX(OutOfStockTable, Stock[SKU ID], ", ")
```  
This measure generates a list of product IDs that are currently out of stock.  

### **2.11 Reorder Point Calculation**  
```DAX
Reorder Point = 
VAR AvgDailySales = [Average Daily Sales]  
VAR AvgLeadTime = AVERAGE(Stock[Average Lead Time (days)])
VAR SafetyStock = [Safety Stock] 
RETURN 
    (AvgDailySales * AvgLeadTime) + SafetyStock
```  
This measure calculates the reorder point based on average daily sales, lead time, and safety stock.  

### **2.12 Reorder Point Alert**  
```DAX
Reorder Point Alert = 
VAR ReorderNeeded = 
    CALCULATE(
        COUNT(Stock[SKU ID]),
        FILTER(Stock, Stock[Current Stock Quantity] <= [Reorder Point])
    )
VAR OutOfStockCount =
    CALCULATE(COUNTROWS(Stock), Stock[Stock Status] = "Out Of Stock")
RETURN
ReorderNeeded - OutOfStockCount
```  
This measure calculates the number of items that need restocking, excluding already out-of-stock items.  

### **2.13 Need Restock Alert**  
```DAX
Need Restock Alert = 
IF( 
     [Total Stock Quantity] <= [Reorder Point], 
    "Restock", 
    "Not Yet" 
)
```  
This measure provides an alert when stock reaches the reorder point.  

### **2.14 Previous Month Name (for reporting)**  
```DAX
Previous Month Name = 
FORMAT(EOMONTH(MAX('Date'[Date]), -1), "MMMM") & ": "
```  
This measure retrieves the name of the previous month for reporting purposes.  

---
## **3. Calculated Columns for ABC Classification & Ranking**  

### **3.1 Percentage Revenue Share**  
```DAX
% Revenue Share = DIVIDE(Stock[Revenue], SUM(Stock[Revenue]))
```  
This column calculates the percentage share of total revenue for each product.  

### **3.2 Cumulative Percentage Revenue**  
```DAX
CP Revenue = 
CALCULATE(
    SUM(Stock[% Revenue Share]),
    FILTER(Stock, Stock[% Revenue Share] >= EARLIER(Stock[% Revenue Share]))
)
```  
This column calculates the cumulative percentage of revenue share, used for ABC classification.  

### **3.3 ABC Classification**  
```DAX
ABC Classification = 
IF(Stock[CP Revenue] <= 0.7, "A", 
   IF(Stock[CP Revenue] <= 0.9, "B", "C"))
```  
This column assigns each product to an ABC classification based on its cumulative revenue share:  
- **A:** High-value, low-volume products (top 70% of revenue)  
- **B:** Moderate-value products (next 20% of revenue)  
- **C:** Low-value, high-volume products (last 10% of revenue)  

### **3.4 Product Ranking by Revenue Share**  
```DAX
Product Rank = RANKX(Stock, Stock[% Revenue Share])
```  
This column ranks products based on their revenue contribution, with **1 being the highest revenue share.**
