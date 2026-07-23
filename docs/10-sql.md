# 10. SQL & Databases - Complete Guide

## 📊 Interview Frequency: 100% | Expected Questions: 20

---

## SQL Query Execution Order

```
Written Order              Actual Execution Order
───────────────              ─────────────────────
1. SELECT                1. FROM
2. FROM                  2. JOIN
3. JOIN                  3. WHERE
4. WHERE                 4. GROUP BY
5. GROUP BY              5. HAVING
6. HAVING                6. SELECT
7. ORDER BY              7. DISTINCT
8. LIMIT                 8. ORDER BY
                         9. LIMIT
```

---

## 1️⃣ Basic CRUD Operations

### CREATE (INSERT):

```sql
-- Single insert
INSERT INTO users (name, email, age)
VALUES ('John Doe', 'john@example.com', 30);

-- Multiple insert
INSERT INTO users (name, email, age)
VALUES 
  ('Jane Doe', 'jane@example.com', 28),
  ('Bob Smith', 'bob@example.com', 35),
  ('Alice Johnson', 'alice@example.com', 29);

-- Insert from another table
INSERT INTO users_archive (name, email, age)
SELECT name, email, age FROM users WHERE age > 60;
```

### READ (SELECT):

```sql
-- Simple select
SELECT * FROM users;
SELECT name, email FROM users;

-- With WHERE
SELECT * FROM users WHERE age > 30;
SELECT * FROM users WHERE name LIKE 'John%';
SELECT * FROM users WHERE age BETWEEN 25 AND 35;
SELECT * FROM users WHERE id IN (1, 2, 3);

-- With ORDER BY
SELECT * FROM users ORDER BY age ASC;
SELECT * FROM users ORDER BY age DESC, name ASC;

-- With LIMIT
SELECT * FROM users LIMIT 10;          -- First 10
SELECT * FROM users LIMIT 10 OFFSET 20; -- Skip 20, take 10 (pagination)
```

### UPDATE:

```sql
-- Update single row
UPDATE users SET age = 31 WHERE id = 1;

-- Update multiple columns
UPDATE users SET age = 31, email = 'newemail@example.com' WHERE id = 1;

-- Update multiple rows
UPDATE users SET status = 'inactive' WHERE last_login < DATE_SUB(NOW(), INTERVAL 6 MONTH);

-- Update with calculation
UPDATE users SET age = age + 1 WHERE birthday = CURDATE();
```

### DELETE:

```sql
-- Delete single row
DELETE FROM users WHERE id = 1;

-- Delete multiple rows
DELETE FROM users WHERE age < 18;

-- Delete all (dangerous!)
DELETE FROM users;  -- ❌ No WHERE clause!

-- Safer: Use archive pattern
INSERT INTO users_archive SELECT * FROM users WHERE age > 65;
DELETE FROM users WHERE age > 65;
```

---

## 2️⃣ JOINs

```
JOIN Types Visualization:

Table A                 Table B
┌────────┌────────┌────────┌
│ ID │ Name  │ ID │ City    │
├────────├────────├────────├
│ 1  │ John  │ 1 │ NYC     │
│ 2  │ Jane  │ 2 │ LA      │
│ 3  │ Bob   │ 3 │ Chicago │
└────────┴────────┴────────┴


INNER JOIN:
┌────────┐        LEFT JOIN:
│ 1  John NYC │        ┌────────┐
│ 2  Jane LA  │        │ 1  John NYC │
│ 3  Bob  Chicago│        │ 2  Jane LA  │
└────────┘        │ 3  Bob  Chicago│
(Only matches)      └────────┘
                    (All from A + matches from B)
```

### INNER JOIN:

```sql
-- Returns rows with matches in both tables
SELECT u.id, u.name, c.city
FROM users u
INNER JOIN cities c ON u.city_id = c.id;

-- Equivalent to
SELECT u.id, u.name, c.city
FROM users u, cities c
WHERE u.city_id = c.id;
```

### LEFT JOIN:

```sql
-- Returns all rows from left table + matches from right
SELECT u.id, u.name, c.city
FROM users u
LEFT JOIN cities c ON u.city_id = c.id;
-- Users without cities will have NULL for city
```

### RIGHT JOIN:

```sql
-- Returns all rows from right table + matches from left
SELECT u.id, u.name, c.city
FROM users u
RIGHT JOIN cities c ON u.city_id = c.id;
-- All cities, even those without users
```

### FULL OUTER JOIN:

```sql
-- All rows from both tables
SELECT u.id, u.name, c.city
FROM users u
FULL OUTER JOIN cities c ON u.city_id = c.id;
-- (Not supported in MySQL, use UNION)

-- Workaround in MySQL
SELECT u.id, u.name, c.city
FROM users u
LEFT JOIN cities c ON u.city_id = c.id
UNION
SELECT u.id, u.name, c.city
FROM users u
RIGHT JOIN cities c ON u.city_id = c.id;
```

### SELF JOIN:

```sql
-- Join table with itself (e.g., employee-manager relationship)
SELECT e.name as employee, m.name as manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

### CROSS JOIN:

```sql
-- Cartesian product (all combinations)
SELECT u.name, c.name
FROM users u
CROSS JOIN cities c;
-- Result: 100 users × 50 cities = 5000 rows
```

---

## 3️⃣ Aggregation Functions

```sql
-- COUNT
SELECT COUNT(*) FROM users;              -- Total rows
SELECT COUNT(email) FROM users;          -- Non-null emails
SELECT COUNT(DISTINCT city) FROM users;  -- Unique cities

-- SUM
SELECT SUM(amount) FROM orders;          -- Total amount
SELECT SUM(quantity) FROM order_items;   -- Total items

-- AVG
SELECT AVG(age) FROM users;              -- Average age
SELECT AVG(price) FROM products;         -- Average price

-- MIN / MAX
SELECT MIN(age), MAX(age) FROM users;    -- Age range
SELECT MIN(price), MAX(price) FROM products;

-- GROUP BY
SELECT city, COUNT(*) as user_count
FROM users
GROUP BY city;

-- GROUP BY with HAVING
SELECT city, COUNT(*) as user_count
FROM users
GROUP BY city
HAVING COUNT(*) > 5;

-- Multiple aggregations
SELECT 
  category,
  COUNT(*) as product_count,
  AVG(price) as avg_price,
  MIN(price) as min_price,
  MAX(price) as max_price
FROM products
GROUP BY category;
```

---

## 4️⃣ Subqueries

```sql
-- Scalar subquery (returns single value)
SELECT * FROM users WHERE age > (SELECT AVG(age) FROM users);

-- IN subquery
SELECT * FROM products
WHERE category_id IN (SELECT id FROM categories WHERE status = 'active');

-- EXISTS subquery
SELECT * FROM users u
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);

-- Subquery in FROM (derived table)
SELECT * FROM (
  SELECT name, age, 
         RANK() OVER (ORDER BY age DESC) as age_rank
  FROM users
) ranked_users
WHERE age_rank <= 10;
```

---

## 5️⃣ Indexing & Query Optimization

### Create Index:

```sql
-- Single column index
CREATE INDEX idx_email ON users(email);

-- Composite index (multiple columns)
CREATE INDEX idx_name_age ON users(name, age);

-- Unique index
CREATE UNIQUE INDEX idx_email_unique ON users(email);

-- Full text index (for text search)
CREATE FULLTEXT INDEX idx_title ON articles(title, content);

-- View indexes
SHOW INDEX FROM users;

-- Drop index
DROP INDEX idx_email ON users;
```

### Query Optimization:

```sql
-- ❌ BAD: Full table scan
SELECT * FROM users WHERE YEAR(created_at) = 2024;

-- ✅ GOOD: Use index
SELECT * FROM users WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';

-- ❌ BAD: Function on indexed column
SELECT * FROM users WHERE LOWER(email) = 'john@example.com';

-- ✅ GOOD: Store in correct case
SELECT * FROM users WHERE email = 'john@example.com';

-- ❌ BAD: OR with non-indexed columns
SELECT * FROM users WHERE age = 30 OR status = 'active';

-- ✅ GOOD: UNION (if columns have indexes)
SELECT * FROM users WHERE age = 30
UNION
SELECT * FROM users WHERE status = 'active';

-- ❌ BAD: LIKE with leading wildcard
SELECT * FROM users WHERE email LIKE '%@example.com';

-- ✅ GOOD: Use specific pattern or FULLTEXT
SELECT * FROM users WHERE email LIKE 'john%';
```

### EXPLAIN Query Execution:

```sql
-- Analyze query performance
EXPLAIN SELECT * FROM users WHERE email = 'john@example.com';

-- Output columns:
-- type: Access method (ALL=full scan, INDEX, RANGE, ref)
-- possible_keys: Indexes that could be used
-- key: Index actually used
-- rows: Estimated rows examined
-- Extra: Additional info

-- Better for detailed analysis
EXPLAIN FORMAT=JSON SELECT * FROM users WHERE email = 'john@example.com'\G
```

---

## 6️⃣ Window Functions (SQL:2003+)

```sql
-- ROW_NUMBER: Unique row numbers
SELECT 
  name,
  salary,
  ROW_NUMBER() OVER (ORDER BY salary DESC) as salary_rank
FROM employees;

-- RANK: With ties
SELECT 
  name,
  salary,
  RANK() OVER (ORDER BY salary DESC) as salary_rank,
  DENSE_RANK() OVER (ORDER BY salary DESC) as dense_rank
FROM employees;

-- PARTITION BY: Group within window
SELECT 
  name,
  department,
  salary,
  RANK() OVER (PARTITION BY department ORDER BY salary DESC) as dept_salary_rank
FROM employees;

-- LAG/LEAD: Previous/next row
SELECT 
  date,
  revenue,
  LAG(revenue) OVER (ORDER BY date) as prev_revenue,
  LEAD(revenue) OVER (ORDER BY date) as next_revenue
FROM daily_sales;

-- Cumulative SUM
SELECT 
  date,
  amount,
  SUM(amount) OVER (ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as cumulative
FROM transactions;
```

---

## 7️⃣ Common Table Expressions (CTE / WITH)

```sql
-- Recursive CTE (for hierarchical data)
WITH RECURSIVE employee_hierarchy AS (
  -- Base case: CEO (no manager)
  SELECT id, name, manager_id, 1 as level
  FROM employees
  WHERE manager_id IS NULL
  
  UNION ALL
  
  -- Recursive case: direct reports
  SELECT e.id, e.name, e.manager_id, eh.level + 1
  FROM employees e
  INNER JOIN employee_hierarchy eh ON e.manager_id = eh.id
)
SELECT id, name, manager_id, level FROM employee_hierarchy;

-- Multiple CTEs
WITH 
  monthly_sales AS (
    SELECT MONTH(date) as month, SUM(amount) as total
    FROM sales
    GROUP BY MONTH(date)
  ),
  avg_sales AS (
    SELECT AVG(total) as avg_monthly
    FROM monthly_sales
  )
SELECT m.month, m.total, a.avg_monthly
FROM monthly_sales m
CROSS JOIN avg_sales a;
```

---

## 8️⃣ Transactions & ACID

```sql
-- Explicit transaction
BEGIN TRANSACTION;  -- or START TRANSACTION

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;  -- Save changes
-- or ROLLBACK;  -- Undo changes

-- Transaction isolation levels
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;  -- Lowest
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;    -- Default in most DBs
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;   -- MySQL default
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;      -- Highest
```

---

## Real-World Example: E-Commerce Queries

```sql
-- User order history with total spent
SELECT 
  u.id,
  u.name,
  COUNT(o.id) as total_orders,
  SUM(o.total) as total_spent,
  AVG(o.total) as avg_order_value,
  MAX(o.created_at) as last_order_date
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name
HAVING total_orders > 0
ORDER BY total_spent DESC;

-- Top 10 selling products with reviews
SELECT 
  p.id,
  p.name,
  COUNT(oi.id) as units_sold,
  SUM(oi.quantity * oi.price) as revenue,
  ROUND(AVG(r.rating), 2) as avg_rating,
  COUNT(r.id) as review_count
FROM products p
LEFT JOIN order_items oi ON p.id = oi.product_id
LEFT JOIN reviews r ON p.id = r.product_id
GROUP BY p.id, p.name
ORDER BY revenue DESC
LIMIT 10;

-- Products not in any order
SELECT * FROM products p
WHERE NOT EXISTS (
  SELECT 1 FROM order_items oi WHERE oi.product_id = p.id
);

-- Month-over-month growth
WITH monthly_revenue AS (
  SELECT 
    DATE_TRUNC(DATE(created_at), MONTH) as month,
    SUM(total) as revenue
  FROM orders
  GROUP BY DATE_TRUNC(DATE(created_at), MONTH)
)
SELECT 
  month,
  revenue,
  LAG(revenue) OVER (ORDER BY month) as prev_month_revenue,
  ROUND(((revenue - LAG(revenue) OVER (ORDER BY month)) / LAG(revenue) OVER (ORDER BY month) * 100), 2) as growth_percent
FROM monthly_revenue;
```

---

## Interview Tips:

✅ **Know the difference between INNER, LEFT, RIGHT, FULL JOINs**

✅ **Understand GROUP BY, HAVING, ORDER BY**

✅ **Know indexing strategies for performance**

✅ **Explain N+1 query problem and solutions**

✅ **Understand transactions and ACID properties**

✅ **Know window functions for ranking/cumulative calculations**

---

📖 **Next Topic:** [React Fundamentals](./11-react.md)
