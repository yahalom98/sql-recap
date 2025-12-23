Got it 👍

---

# SQL for Beginners (Excel Users)

120-Minute Hands-On Session

Target audience:
People who know Excel (filters, formulas, pivot tables, VLOOKUP/XLOOKUP)
No prior SQL knowledge required

---

## Session Agenda (120 minutes)

| Time    | Topic                         |
| ------- | ----------------------------- |
| 0–10    | Excel to SQL mindset          |
| 10–25   | Database and tables           |
| 25–40   | SELECT and WHERE              |
| 40–55   | ORDER BY, LIMIT, calculations |
| 55–75   | GROUP BY (Pivot tables)       |
| 75–100  | JOIN (VLOOKUP/XLOOKUP)        |
| 100–110 | LEFT JOIN and NULL            |
| 110–120 | Final report and recap        |

---

## 1. Excel to SQL Mapping (10 min)

| Excel             | SQL      |
| ----------------- | -------- |
| Sheet / Table     | Table    |
| Column            | Column   |
| Row               | Record   |
| Filter            | WHERE    |
| Sort              | ORDER BY |
| Pivot Table       | GROUP BY |
| VLOOKUP / XLOOKUP | JOIN     |

---

## 2. Database Setup (15 min)

Copy and paste everything below into your SQL editor.

```sql
DROP TABLE IF EXISTS order_items;
DROP TABLE IF EXISTS orders;
DROP TABLE IF EXISTS products;
DROP TABLE IF EXISTS customers;

CREATE TABLE customers (
  customer_id INT PRIMARY KEY,
  full_name VARCHAR(100),
  city VARCHAR(50),
  email VARCHAR(100)
);

CREATE TABLE products (
  product_id INT PRIMARY KEY,
  product_name VARCHAR(100),
  category VARCHAR(50),
  price NUMERIC(10,2)
);

CREATE TABLE orders (
  order_id INT PRIMARY KEY,
  customer_id INT,
  order_date DATE,
  status VARCHAR(20)
);

CREATE TABLE order_items (
  order_item_id INT PRIMARY KEY,
  order_id INT,
  product_id INT,
  quantity INT,
  unit_price NUMERIC(10,2)
);
```

### Insert Sample Data

```sql
INSERT INTO customers VALUES
(1,'Dana Levi','Tel Aviv','dana@gmail.com'),
(2,'Omer Cohen','Haifa','omer@gmail.com'),
(3,'Noa Shani','Jerusalem','noa@gmail.com'),
(4,'Avi Mizrahi','Tel Aviv',NULL);

INSERT INTO products VALUES
(10,'Wireless Mouse','Accessories',79.90),
(11,'Keyboard','Accessories',349.00),
(12,'Monitor','Screens',999.00),
(13,'USB Cable','Accessories',29.90);

INSERT INTO orders VALUES
(100,1,'2025-12-01','PAID'),
(101,2,'2025-12-02','PAID'),
(102,2,'2025-12-05','CANCELLED'),
(103,3,'2025-12-06','PAID');

INSERT INTO order_items VALUES
(1,100,10,1,79.90),
(2,100,13,2,29.90),
(3,101,12,1,999.00),
(4,102,11,1,349.00),
(5,103,13,3,29.90);
```

---

## 3. SELECT Basics (15 min)

```sql
SELECT * FROM customers;

SELECT full_name, city
FROM customers;
```

### Rename Columns

```sql
SELECT
  full_name AS name,
  city AS location
FROM customers;
```

### Exercise

Show only product_name and price.

<details>
<summary>Solution</summary>

```sql
SELECT product_name, price
FROM products;
```

</details>

---

## 4. Filtering with WHERE (15 min)

### Basic filter

```sql
SELECT *
FROM customers
WHERE city = 'Tel Aviv';
```

### Numeric conditions

```sql
SELECT *
FROM products
WHERE price >= 100;
```

### AND / OR

```sql
SELECT *
FROM orders
WHERE status = 'PAID'
  AND order_date >= '2025-12-02';
```

### IN and LIKE

```sql
SELECT *
FROM customers
WHERE city IN ('Tel Aviv','Haifa');

SELECT *
FROM products
WHERE product_name LIKE '%USB%';
```

### Exercise

Accessories cheaper than 100.

<details>
<summary>Solution</summary>

```sql
SELECT *
FROM products
WHERE category = 'Accessories'
  AND price < 100;
```

</details>

---

## 5. Sorting and Calculations (15 min)

### Sorting

```sql
SELECT *
FROM products
ORDER BY price DESC;
```

### Top N

```sql
SELECT *
FROM products
ORDER BY price ASC
LIMIT 2;
```

### Calculated Columns

```sql
SELECT
  product_name,
  price,
  ROUND(price * 1.17, 2) AS price_with_vat
FROM products;
```

---

## 6. GROUP BY (Pivot Tables) (20 min)

### Total per order

```sql
SELECT
  order_id,
  SUM(quantity * unit_price) AS order_total
FROM order_items
GROUP BY order_id;
```

### Count orders by status

```sql
SELECT
  status,
  COUNT(*) AS orders_count
FROM orders
GROUP BY status;
```

### HAVING

```sql
SELECT
  order_id,
  SUM(quantity * unit_price) AS total
FROM order_items
GROUP BY order_id
HAVING SUM(quantity * unit_price) > 200;
```

### Exercise

Revenue per product.

<details>
<summary>Solution</summary>

```sql
SELECT
  product_id,
  SUM(quantity * unit_price) AS revenue
FROM order_items
GROUP BY product_id
ORDER BY revenue DESC;
```

</details>

---

## 7. JOIN (VLOOKUP/XLOOKUP) (25 min)

### Orders with customers

```sql
SELECT
  o.order_id,
  o.order_date,
  c.full_name,
  o.status
FROM orders o
JOIN customers c
  ON c.customer_id = o.customer_id;
```

### Full sales report

```sql
SELECT
  o.order_id,
  c.full_name,
  p.product_name,
  oi.quantity,
  oi.unit_price,
  oi.quantity * oi.unit_price AS line_total
FROM order_items oi
JOIN orders o ON o.order_id = oi.order_id
JOIN customers c ON c.customer_id = o.customer_id
JOIN products p ON p.product_id = oi.product_id
WHERE o.status = 'PAID';
```

### Exercise

Paid orders per customer.

<details>
<summary>Solution</summary>

```sql
SELECT
  c.full_name,
  COUNT(*) AS paid_orders
FROM orders o
JOIN customers c ON c.customer_id = o.customer_id
WHERE o.status = 'PAID'
GROUP BY c.full_name;
```

</details>

---

## 8. LEFT JOIN and NULL (10 min)

### Keep all customers

```sql
SELECT
  c.full_name,
  COUNT(o.order_id) AS orders_count
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.customer_id
GROUP BY c.full_name;
```

### NULL handling

```sql
SELECT
  full_name,
  COALESCE(email, 'NO EMAIL') AS email
FROM customers;
```

---

## 9. Final Report (10 min)

Revenue per customer (paid orders only):

```sql
SELECT
  c.customer_id,
  c.full_name,
  SUM(oi.quantity * oi.unit_price) AS total_revenue
FROM customers c
JOIN orders o ON o.customer_id = c.customer_id
JOIN order_items oi ON oi.order_id = o.order_id
WHERE o.status = 'PAID'
GROUP BY c.customer_id, c.full_name
ORDER BY total_revenue DESC;
```

---

---

<!-- EXAM -->
Below is a **clean SQL exam**, **Markdown format**, **no icons**, suitable for **beginners who know Excel**.
You can paste this directly into **`EXAM.md`** in GitHub.

I’ve structured it so it can be done in **90–120 minutes**.

---

# SQL Exam – Beginner Level (Excel Background)

**Duration:** 120 minutes
**Allowed material:** None
**Database:** Use the database provided in class (customers, products, orders, order_items)
**Instructions:**

* Write SQL queries only
* Do not modify table structures
* Do not delete data
* Order results when it makes sense

---

## Tables Overview

* customers(customer_id, full_name, city, email)
* products(product_id, product_name, category, price)
* orders(order_id, customer_id, order_date, status)
* order_items(order_item_id, order_id, product_id, quantity, unit_price)

---

## Part A – Basic SELECT and WHERE (20 points)

### Question 1 (5 points)

Display all customers who live in **Tel Aviv**.
Show all columns.

---

### Question 2 (5 points)

Display the **product name and price** of all products that cost **more than 100**.

---

### Question 3 (5 points)

Display all orders that have status **PAID** and were created **after 2025-12-01**.

---

### Question 4 (5 points)

Display all products whose name contains the word **USB**.

---

## Part B – Sorting and Calculations (20 points)

### Question 5 (5 points)

Display all products sorted by **price from highest to lowest**.

---

### Question 6 (5 points)

Display the **two cheapest products**.

---

### Question 7 (10 points)

Display:

* product_name
* price
* a calculated column called `price_with_vat` (price * 1.17, rounded to 2 decimals)

---

## Part C – GROUP BY and Aggregations (25 points)

### Question 8 (10 points)

For each order, display:

* order_id
* total order amount (quantity * unit_price)

---

### Question 9 (5 points)

Display how many orders exist for each **order status**.

---

### Question 10 (10 points)

Display only the orders whose **total amount is greater than 200**.

---

## Part D – JOINs (35 points)

### Question 11 (10 points)

Display all orders with:

* order_id
* order_date
* customer full name
* order status

---

### Question 12 (10 points)

Display a full sales report with:

* order_id
* customer full name
* product name
* quantity
* unit price
* line total (quantity * unit_price)

Include **only PAID orders**.

---

### Question 13 (10 points)

For each customer, display:

* customer full name
* number of PAID orders

---

### Question 14 (5 points)

Display all customers who **never placed an order**.

---

## Part E – Final Business Report (Bonus – 10 points)

### Question 15 (Bonus)

Display a report showing:

* customer_id
* customer full name
* total revenue generated by the customer

Only include **PAID orders**, sorted from highest revenue to lowest.

---

---

---


