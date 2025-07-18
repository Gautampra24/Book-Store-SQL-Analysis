
# Book Store SQL Analysis

**Book Store SQL Analysis** repository. This project provides a complete walkthrough of setting up a book store database and performing meaningful SQL queries to extract business insights. From inserting data to deep analytical queries, every step is documented below.

---

## Database Setup

### 1. Create Tables

```sql
CREATE TABLE books (
  Book_ID SERIAL PRIMARY KEY,
  Title VARCHAR(100),
  Author VARCHAR(100),
  Genre VARCHAR(100),
  Published_Year INT,
  Price NUMERIC(10,2),
  Stock INT
);

CREATE TABLE customers (
  Customer_ID SERIAL PRIMARY KEY,
  Name VARCHAR(50),
  Email VARCHAR(100),
  Phone VARCHAR(20),
  City VARCHAR(100),
  Country VARCHAR(100)
);

CREATE TABLE orders (
  Order_ID SERIAL PRIMARY KEY,
  Customer_ID INT REFERENCES customers(Customer_ID),
  Book_ID INT REFERENCES books(Book_ID),
  Order_Date DATE,
  Quantity INT,
  Total_Amount INT
);

-- Modify column type
ALTER TABLE orders
ALTER COLUMN Total_Amount TYPE NUMERIC (10,2);
```

---

### 2. Import Data (Using `COPY`)

```sql
-- Books
COPY books(Book_ID, Title, Author, Genre, Published_Year, Price, Stock)
FROM 'E:\my SQL\ST - SQL ALL PRACTICE FILES\All Excel Practice Files\Books.csv'
CSV HEADER;

-- Customers
COPY customers(Customer_ID, Name, Email, Phone, City, Country)
FROM 'E:\my SQL\ST - SQL ALL PRACTICE FILES\All Excel Practice Files\Customers.csv'
CSV HEADER;

-- Orders
COPY orders(Order_ID, Customer_ID, Book_ID, Order_Date, Quantity, Total_Amount)
FROM 'E:\my SQL\ST - SQL ALL PRACTICE FILES\All Excel Practice Files\Orders.csv'
CSV HEADER;
```

---

## Data Analysis & Business Insights

### 1. All Fiction Books

```sql
SELECT * FROM books
WHERE genre = 'Fiction';
```

### 2. Books Published After 1950

```sql
SELECT * FROM books
WHERE Published_Year > 1950;
```

<details>
<summary>🧠 Other Methods to Write This Query</summary>

* Using `BETWEEN`:

  ```sql
  SELECT * FROM books
  WHERE Published_Year BETWEEN 1951 AND 2025;
  ```

* Using `NOT <=`:

  ```sql
  SELECT * FROM books
  WHERE NOT Published_Year <= 1950;
  ```

* Using CTE:

  ```sql
  WITH recent_books AS (
      SELECT * FROM books WHERE Published_Year > 1950
  )
  SELECT * FROM recent_books;
  ```

* Create a View:

  ```sql
  CREATE VIEW recent_books AS
  SELECT * FROM books WHERE Published_Year > 1950;

  SELECT * FROM recent_books;
  ```

</details>

### 3. Customers from Canada

```sql
SELECT * FROM customers
WHERE Country = 'Canada';
```

Alternative using View:

```sql
CREATE VIEW Canadian_cus AS
SELECT * FROM customers
WHERE Country = 'Canada';

SELECT * FROM Canadian_cus;
```

---

### 4. Orders Placed in November 2023

```sql
SELECT * FROM orders
WHERE Order_Date BETWEEN '2023-11-01' AND '2023-11-30';
```

Alternative using `TO_CHAR()`:

```sql
SELECT * FROM orders
WHERE TO_CHAR(Order_Date, 'YYYY-MM') = '2023-11';
```

---

### 5. Total Stock of Books Available

```sql
SELECT SUM(Stock) AS Books_Stock FROM books;
```

---

### 6. Most Expensive Book

```sql
SELECT * FROM books
ORDER BY Price DESC
LIMIT 1;
```

---

### 7. Customers Who Ordered More Than 1 Quantity

```sql
SELECT c.Name AS Customer_Name, o.Quantity AS Purchase_Quantity
FROM Customers c
JOIN orders o ON c.Customer_ID = o.Customer_ID
WHERE o.Quantity > 1;
```

---

### 8. Orders With Total Amount > \$20

```sql
SELECT c.Name AS Customer_Name, o.Quantity AS Quantity_Purchased, o.Total_Amount
FROM customers c
JOIN orders o ON c.Customer_ID = o.Customer_ID
WHERE o.Total_Amount > 20
ORDER BY o.Total_Amount DESC;
```

---

### 9. List of All Book Genres

```sql
SELECT DISTINCT Genre FROM books;
```

---

### 10. Book with Lowest Stock

```sql
SELECT * FROM books
ORDER BY Stock
LIMIT 1;
```

---

### 11. Total Revenue Generated

```sql
SELECT SUM(Total_Amount) AS Total_Revenue FROM orders;
```

---

## 🔍 Deeper Insights

### 12. Total Books Sold by Genre

```sql
SELECT b.Genre AS Book_Category, SUM(o.Quantity) AS Sold_Quantity
FROM books b
JOIN orders o ON b.Book_ID = o.Book_ID
GROUP BY b.Genre;
```

---

### 13. Average Price of Fantasy Books

```sql
SELECT AVG(Price) AS Avg_Fantasy_Price FROM books
WHERE Genre = 'Fantasy';
```

---

### 14. Customers With at Least 2 Orders

```sql
SELECT c.Name, COUNT(o.Order_ID) AS Order_Count
FROM customers c
JOIN orders o ON c.Customer_ID = o.Customer_ID
GROUP BY c.Name
HAVING COUNT(o.Order_ID) >= 2;
```

---

### 15. Most Frequently Ordered Book

```sql
SELECT b.Title AS Book_Name, SUM(o.Quantity) AS Total_Ordered
FROM books b
JOIN orders o ON b.Book_ID = o.Book_ID
GROUP BY b.Title
ORDER BY Total_Ordered DESC
LIMIT 1;
```

---

### 16. Top 3 Most Expensive Fantasy Books

```sql
SELECT * FROM books
WHERE Genre = 'Fantasy'
ORDER BY Price DESC
LIMIT 3;
```

---

### 17. Total Quantity Sold by Each Author

```sql
SELECT b.Author AS Author_Name, SUM(o.Quantity) AS Total_Sold
FROM books b
JOIN orders o ON b.Book_ID = o.Book_ID
GROUP BY b.Author;
```

---

### 18. Cities Where Customers Spent Over \$30

```sql
SELECT DISTINCT c.City, c.Name AS Customer_Name, o.Total_Amount
FROM customers c
JOIN orders o ON c.Customer_ID = o.Customer_ID
WHERE o.Total_Amount > 30;
```

---

### 19. Customer Who Spent the Most

```sql
SELECT c.Name AS Customer_Name, o.Total_Amount
FROM customers c
JOIN orders o ON c.Customer_ID = o.Customer_ID
ORDER BY o.Total_Amount DESC
LIMIT 1;
```

---

### 20. Remaining Stock After Orders

```sql
SELECT b.Book_ID, b.Title, b.Stock, 
       COALESCE(SUM(o.Quantity), 0) AS Sold_Quantity,
       b.Stock - COALESCE(SUM(o.Quantity), 0) AS Remaining_Stock
FROM books b
LEFT JOIN orders o ON b.Book_ID = o.Book_ID
GROUP BY b.Book_ID;
```

---

## 📁 Tables Preview

```sql
SELECT * FROM books;
SELECT * FROM customers;
SELECT * FROM orders;
```

---

## ✅ Conclusion

This SQL analysis gives a complete overview of:

* Sales trends
* Customer behavior
* Inventory levels
* Revenue performance

It can help stakeholders make strategic business decisions in pricing, stock management, and customer targeting.

