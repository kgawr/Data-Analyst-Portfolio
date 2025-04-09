# Ecommerce Portfolio Project

E-commerce Data Analysis Project

This project demonstrates the end-to-end process of creating a data analysis solution for an e-commerce business, from data generation to visualization.

Key Steps:

    Data Generation:
        Created a Python script to generate realistic e-commerce data
        Produced datasets for customers, products, orders, and order details

    Database Setup:
        Designed and implemented a PostgreSQL database schema
        Created tables for customers, products, orders, and order details
        Imported generated data into the database

    SQL Analysis:
        Developed SQL queries for key business metrics
        Analyzed sales trends, customer behavior, and product performance

    Power BI Integration:
        Connected Power BI to the PostgreSQL database
        Created a date table for time intelligence functions

    DAX Measures:
        Developed DAX measures for advanced metrics
        Calculated KPIs such as Monthly Retention Rate, Customer Churn Rate, and New Customer Acquisition

    Visualization:
        Designed an interactive dashboard in Power BI
        Created visualizations for sales trends, customer retention, and product performance
        Implemented slicers and filters for dynamic data exploration

    Insights and Recommendations:
        Interpreted the visualized data to derive business insights
        Formulated data-driven recommendations for improving sales and customer retention

This project showcases skills in data generation, SQL, DAX, and data visualization, demonstrating the ability to transform raw data into actionable business insights.


# Generating random data

import random
from datetime import datetime, timedelta
import csv
from faker import Faker

fake = Faker()

Set random seed for reproducibility
random.seed(42)
fake.seed_instance(42)

Generate Customers

def generate_customers(num_customers):
    customers = []
    for i in range(1, num_customers + 1):
        customers.append({
            'CustomerID': i,
            'FirstName': fake.first_name(),
            'LastName': fake.last_name(),
            'Country': fake.country(),
            'City': fake.city(),
            'Email': fake.email(),
            'JoinDate': fake.date_between(start_date='-3y', end_date='today')
        })
    return customers

Generate Products

def generate_products(num_products):
    categories = ['Electronics', 'Clothing', 'Home & Garden', 'Books', 'Sports']
    subcategories = {
        'Electronics': ['Smartphones', 'Laptops', 'Cameras', 'Accessories'],
        'Clothing': ['Shirts', 'Pants', 'Dresses', 'Shoes'],
        'Home & Garden': ['Furniture', 'Decor', 'Kitchen', 'Gardening'],
        'Books': ['Fiction', 'Non-fiction', 'Educational', 'Children'],
        'Sports': ['Equipment', 'Apparel', 'Fitness', 'Outdoor']
    }
    
    products = []
    for i in range(1, num_products + 1):
        category = random.choice(categories)
        products.append({
            'ProductID': i,
            'ProductName': fake.catch_phrase(),
            'Category': category,
            'SubCategory': random.choice(subcategories[category]),
            'UnitPrice': round(random.uniform(10, 1000), 2),
            'StockLevel': random.randint(0, 1000)
        })
    return products

Generate Orders and Order Details

def generate_orders_and_details(num_orders, customers, products):
    orders = []
    order_details = []
    order_detail_id = 1
    
    for i in range(1, num_orders + 1):
        order_date = fake.date_between(start_date='-1y', end_date='today')
        ship_date = order_date + timedelta(days=random.randint(1, 14))
        
        orders.append({
            'OrderID': i,
            'CustomerID': random.choice(customers)['CustomerID'],
            'OrderDate': order_date,
            'ShipDate': ship_date,
            'ShipMode': random.choice(['Standard', 'Express', 'Next Day'])
        })
        
         Generate 1 to 5 order details for each order
        for _ in range(random.randint(1, 5)):
            order_details.append({
                'OrderDetailID': order_detail_id,
                'OrderID': i,
                'ProductID': random.choice(products)['ProductID'],
                'Quantity': random.randint(1, 10),
                'Discount': round(random.uniform(0, 0.2), 2)
            })
            order_detail_id += 1
    
    return orders, order_details

Generate data

num_customers = 1000
num_products = 200
num_orders = 10000

customers = generate_customers(num_customers)
products = generate_products(num_products)
orders, order_details = generate_orders_and_details(num_orders, customers, products)

Write data to CSV files
 
def write_to_csv(data, filename):
    with open(filename, 'w', newline='') as csvfile:
        writer = csv.DictWriter(csvfile, fieldnames=data[0].keys())
        writer.writeheader()
        for row in data:
            writer.writerow(row)

write_to_csv(customers, 'customers.csv')
write_to_csv(products, 'products.csv')
write_to_csv(orders, 'orders.csv')
write_to_csv(order_details, 'order_details.csv')

print("Data generation complete. CSV files have been created.")


# PostgreSQL

Sales by Month
 
SELECT 
    DATE_TRUNC('month', o.OrderDate) as Month,
    SUM(od.Quantity * p.UnitPrice * (1-od.Discount))::numeric(10,2) as Revenue
FROM Orders o
JOIN OrderDetails od ON o.OrderID = od.OrderID
JOIN Products p ON od.ProductID = p.ProductID
GROUP BY DATE_TRUNC('month', o.OrderDate)
ORDER BY Month;

Sales Last 30 Days
 
SELECT 
    SUM(od.Quantity * p.UnitPrice * (1-od.Discount))::numeric(10,2) as Revenue
FROM Orders o
JOIN OrderDetails od ON o.OrderID = od.OrderID
JOIN Products p ON od.ProductID = p.ProductID
WHERE o.OrderDate >= CURRENT_DATE - INTERVAL '30 days';

Monthly Growth Rate
 
WITH MonthlyRevenue AS (
    SELECT 
        DATE_TRUNC('month', o.OrderDate) as Month,
        SUM(od.Quantity * p.UnitPrice * (1-od.Discount))::numeric(10,2) as Revenue
    FROM Orders o
    JOIN OrderDetails od ON o.OrderID = od.OrderID
    JOIN Products p ON od.ProductID = p.ProductID
    GROUP BY DATE_TRUNC('month', o.OrderDate)
)
SELECT 
    Month,
    Revenue,
    LAG(Revenue) OVER (ORDER BY Month) as PreviousMonthRevenue,
    CASE 
        WHEN LAG(Revenue) OVER (ORDER BY Month) = 0 THEN NULL
        WHEN LAG(Revenue) OVER (ORDER BY Month) IS NULL THEN NULL
        ELSE ROUND(
            ((Revenue - LAG(Revenue) OVER (ORDER BY Month)) * 100.0 / 
            LAG(Revenue) OVER (ORDER BY Month))::numeric,
            2
        )
    END as GrowthRate
FROM MonthlyRevenue
ORDER BY Month;

Top Selling Products
 
SELECT 
    p.ProductName,
    p.Category,
    SUM(od.Quantity) as TotalQuantitySold,
    SUM(od.Quantity * p.UnitPrice * (1-od.Discount)) as TotalRevenue
FROM Products p
JOIN OrderDetails od ON p.ProductID = od.ProductID
GROUP BY p.ProductName, p.Category
ORDER BY TotalRevenue DESC;

Lifetime value
 
SELECT 
    c.CustomerID,
    c.FirstName,
    c.LastName,
    COUNT(DISTINCT o.OrderID) as TotalOrders,
    SUM(od.Quantity * p.UnitPrice * (1-od.Discount)) as LifetimeValue
FROM Customers c
JOIN Orders o ON c.CustomerID = o.CustomerID
JOIN OrderDetails od ON o.OrderID = od.OrderID
JOIN Products p ON od.ProductID = p.ProductID
GROUP BY c.CustomerID, c.FirstName, c.LastName
ORDER BY LifetimeValue DESC;

Retention rate
 
WITH PreviousCustomers AS (
    SELECT DISTINCT CustomerID
    FROM Orders
    WHERE OrderDate BETWEEN 
        CURRENT_DATE - INTERVAL '2 months' 
        AND CURRENT_DATE - INTERVAL '1 month'
),
RetainedCustomers AS (
    SELECT DISTINCT CustomerID
    FROM Orders
    WHERE OrderDate >= CURRENT_DATE - INTERVAL '1 month'
    AND CustomerID IN (SELECT CustomerID FROM PreviousCustomers)
)
SELECT 
    ROUND(
        (COUNT(DISTINCT RetainedCustomers.CustomerID) * 100.0 / 
        NULLIF(COUNT(DISTINCT PreviousCustomers.CustomerID), 0))::numeric, 
        2
    ) as RetentionRate
FROM PreviousCustomers
LEFT JOIN RetainedCustomers ON 1=1;

Avg shipping days
 
SELECT 
    ShipMode,
    ROUND(AVG(ShipDate - OrderDate)::numeric, 2) as AvgShippingDays,
    COUNT(*) as TotalOrders
FROM Orders
GROUP BY ShipMode;
