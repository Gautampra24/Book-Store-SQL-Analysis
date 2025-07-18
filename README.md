# Book-Store-SQL-Analysis
Book Store SQL Analysis. Here we have solved multiple questions, and solve all queries for book store owner. So let's go brekdown. 


CREATE TABLE books(
Book_ID SERIAL PRIMARY KEY,
Title VARCHAR(100),
Author VARCHAR(100),
Genre VARCHAR(100),
Published_Year INT,
Price NUMERIC(10,2),
Stock INT
);

CREATE TABLE customers(
Customer_ID SERIAL PRIMARY KEY,
Name VARCHAR(50),
Email VARCHAR(100),
Phone VARCHAR(20),
City VARCHAR(100),
Country VARCHAR(100)
);

CREATE TABLE orders(
Order_ID SERIAL PRIMARY KEY,
Customer_ID INT REFERENCES customers(Customer_ID),
Book_ID INT REFERENCES books(Book_ID),
Order_Date DATE,
Quantity INT,
Total_Amount INT 
);

ALTER TABLE orders
ALTER COLUMN Total_Amount TYPE NUMERIC (10,2);



--To Insearting data into table frome the local divice.
--TO BOOKS

COPY books(Book_ID, Title, Author, Genre, Published_Year, Price, Stock)
FROM 'E:\my SQL\ST - SQL ALL PRACTICE FILES\All Excel Practice Files\Books.csv'
CSV HEADER;

--to customers
COPY customers(Customer_ID, Name, Email, Phone, City,Country)
FROM 'E:\my SQL\ST - SQL ALL PRACTICE FILES\All Excel Practice Files\Customers.csv'
CSV HEADER;

--TO Orders
COPY orders(Order_ID, Customer_ID, Book_ID, Order_Date, Quantity, Total_Amount)
FROM 'E:\my SQL\ST - SQL ALL PRACTICE FILES\All Excel Practice Files\Orders.csv'
CSV HEADER;

---- Now analize this database to get a insight that requred for our company stackholder.

-- 1) Retrieve all books in the "Fiction" genre; ✅✅

SELECT*FROM books
WHERE genre = 'Fiction';

-- 2) Find books published after the year 1950 ✅✅

SELECT*FROM books
WHERE Published_Year >1950;


--is one way to retrieve records from the `books` table where the `Published_Year` is greater than 1950.

--You can write the **same logic in several alternative ways**, depending on the use case. Here are a few examples:

---

--✅ 1. **Using `BETWEEN` (if you want a range)**

```sql
SELECT * FROM books
WHERE Published_Year BETWEEN 1951 AND 2025;
```

--This retrieves all books published **after 1950** and up to 2025.

--✅ 2. **Using `NOT` and `<=`**

```sql
SELECT * FROM books
WHERE NOT Published_Year <= 1950;
```

--✅ 3. **Using `EXISTS` with a subquery** *(not efficient, but possible)*:

```sql
SELECT * FROM books b
WHERE EXISTS (
    SELECT 1 FROM books
    WHERE Published_Year > 1950 AND id = b.id
);

--✅ 4. **Using a Common Table Expression (CTE)**

```sql
WITH recent_books AS (
    SELECT * FROM books
    WHERE Published_Year > 1950
)
SELECT * FROM recent_books;
```
--✅ 5. **Using a View (for repeated use)**

Create a view:

```sql
CREATE VIEW recent_books AS
SELECT * FROM books
WHERE Published_Year > 1950;
```

Then:

```sql
SELECT * FROM recent_books;
```

---
--✅ 6. **Using `FILTER` clause in aggregates** (for summaries):

SELECT COUNT(*) FILTER (WHERE Published_Year > 1950) AS recent_books_count
FROM books;


---✅ 7. **Using a function (if needed in application logic)**:

--You could define a function like:


CREATE OR REPLACE FUNCTION get_recent_books()
RETURNS TABLE(id INT, title TEXT, Published_Year INT) AS $$
BEGIN
    RETURN QUERY
    SELECT id, title, Published_Year FROM books
    WHERE Published_Year > 1950;
END;
$$ LANGUAGE plpgsql;


-- Then call:

SELECT * FROM get_recent_books();



-- 3) List all customers from the Canada ✅
SELECT*FROM customers
WHERE Country ='Canada';

--method 2

CREATE VIEW Canadian_cus AS
	SELECT*FROM customers
	WHERE Country ='Canada';

SELECT*FROM Canadian_cus;

--4) Show orders placed in November 2023

SELECT*FROM orders
WHERE Order_Date BETWEEN '01-11-2023' AND '30-11-2023';

--METHOD 2 ✅  Using TO_CHAR()

SELECT * FROM orders
WHERE TO_CHAR(Order_Date, 'YYYY-MM') = '2023-11';

-- METHOD 3 Using EXTRACT function
SELECT * FROM orders
WHERE EXTRACT(MONTH FROM Order_Date) = 11
  AND EXTRACT(YEAR FROM Order_Date) = 2023;

-- 5) Retrieve the total stock of books available
SELECT SUM(Stock) AS Books_Stock FROM books;

--6) Find the details of the most expensive book

SELECT * FROM books
ORDER BY Price DESC
LIMIT 20;

--7) Show all customers who ordered more than 1 quantity of a book
SELECT c.Name AS Customer_Name, o.Quantity AS Purchese_Qn 
FROM Customers c
JOIN orders o ON c.Customer_ID = o.Customer_ID
WHERE o.Quantity > 1;

--other one method
SELECT*FROM orders
WHERE Quantity>1;


--8) Retrieve all orders where the total amount exceeds $20

SELECT c.Name AS Customer_Name, o.Quantity AS Purchesed_QN, o.Total_Amount
FROM customers c
JOIN orders o ON c.Customer_ID = o.Customer_ID
WHERE o.Total_Amount>20
ORDER BY o.Total_Amount DESC;

-- 9) List all genres available in the Books table

SELECT Genre FROM books;

-- 10) Find the book with the lowest stock

SELECT*FROM books
ORDER BY Stock;

--11) Calculate the total revenue generated from all orders
SELECT SUM(Total_Amount) AS Total_Generated_rev FROM orders;


-------------------Now Let's Find deep details from this data-----------

--12) Retrieve the total number of books sold for each genre

SELECT b.Genre AS Book_Categories, SUM(o.Quantity) AS Sold_QN
FROM books b
JOIN orders o ON b.Book_ID = o.Book_ID
GROUP BY b.Genre;


SELECT*FROM books;
SELECT*FROM customers;
SELECT*FROM orders;

--13) Find the average price of books in the "Fantasy" genre

SELECT AVG(Price) AS Average_pc_Fantasy FROM books
WHERE Genre = 'Fantasy';

--14) List customers who have placed at least 2 orders

SELECT c.Name AS Customer_Name, o.Quantity AS Orderd_Placed
FROM customers c
JOIN orders o ON c.Customer_ID = o.Customer_ID
WHERE o.Quantity>=2;


--15)Find the most frequently ordered book

SELECT b.Title AS Book_Name, o.Quantity AS Higher_Sold
FROM books b
INNER JOIN orders o ON o.Book_ID = b.Book_ID
ORDER BY o.Quantity DESC;

--16)Show the top 3 most expensive books of 'Fantasy' Genre

SELECT*FROM books
WHERE Genre ='Fantasy'
ORDER BY Price DESC
LIMIT 3;

--17) Retrieve the total quantity of books sold by each author

SELECT b.Author AS Author_Name, SUM(o.Quantity) AS Sold_Books
FROM books b
INNER JOIN orders o ON b.Book_ID = o.Book_ID
GROUP BY b.Author;

--18)List the cities where customers who spent over $30 are located

SELECT c.City AS Customer_City, c.Name AS Customer_Nanme, o.Total_Amount AS Total_Expences
FROM customers c
JOIN orders o ON c.Customer_ID = o.Customer_ID
WHERE o.Total_Amount>30;

--19)Find the customer who spent the most on orders

SELECT c.Name AS Customer_Nanme, o.Total_Amount AS Highest_Expences
FROM customers c
JOIN orders o ON c.Customer_ID = o.Customer_ID
ORDER BY o.Total_Amount DESC ;

--20) Calculate the stock remaining after fulfilling all orders

SELECT SUM(Stock) from books;
SELECT SUM(Quantity) from orders;

SELECT b.book_id, b.title, b.stock, COALESCE(SUM(O.Quantity),0) AS Sold_Quantity,
b.stock - COALESCE(SUM(O.Quantity),0) AS Remaining_Stocks
FROM books b
LEFT JOIN orders o ON b.book_id = o.book_id
GROUP BY b.book_id;









