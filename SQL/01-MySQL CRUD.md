# MySQL CRUD

# MySQL CRUD Operations

## 1. CREATE - INSERT Operations

### Basic INSERT

```
*- Insert a single row*
INSERT INTO users (username, email, created_at)
VALUES ('john_doe', 'john@example.com', NOW());
```

### INSERT with Multiple Rows (Bulk Insert)

```
*- Bulk insert (much faster than individual inserts)*
INSERT INTO log_entries (user_id, action, created_at)
VALUES (1, 'login', NOW()), (2, 'purchase', NOW()), (3, 'update', NOW()), (4, 'login', NOW()), (5, 'logout', NOW());
```

### INSERT or UPDATE (Upsert)

```
*- Insert if not exists, update if exists (requires unique/primary key)*
INSERT INTO users (id, email, name, updated_at)
VALUES (1, 'user@example.com', 'New User', NOW())
ON DUPLICATE KEY UPDATE name = VALUES(name), updated_at = VALUES(updated_at);
```

## 2. READ - SELECT Operations

### Basic SELECT

```
*- Select all columns from a table*
SELECT * FROM users;
*- Select specific columns*
SELECT id, username, email FROM users;
```

### SELECT with WHERE Clause

```
*- Filter results with conditions*
SELECT * FROM users WHERE status = 'active';
*- Multiple conditions*
SELECT * FROM orders
WHERE status = 'completed'
AND created_at > '2023-01-01';
```

### SELECT with JOIN

```
*- Join related tables*
SELECT u.id, u.username, o.order_id, o.total
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE u.status = 'active';
```

### SELECT with Aggregation

```
*- Count, sum, average, etc.*
SELECT COUNT(*) as total_orders, SUM(total) as revenue, AVG(total) as average_order_value
FROM orders
WHERE status = 'completed';
```

## 3. UPDATE Operations

### Basic UPDATE

```
*- Update a single row*
UPDATE users
SET status = 'inactive'
WHERE id = 5;
```

### UPDATE Multiple Rows

```
*- Update multiple rows*
UPDATE users
SET status = 'inactive'
WHERE last_login < DATE_SUB(NOW(), INTERVAL 90 DAY);
```

### Bulk UPDATE

```
*- Bulk update with multiple IDs*
UPDATE users
SET status = 'inactive'
WHERE id IN (1, 5, 10, 15, 20);
```

### UPDATE with JOIN

```
*- Update based on data in another table*
UPDATE orders o
JOIN users u ON o.user_id = u.id
SET o.status = 'canceled'
WHERE u.status = 'suspended';
```

## 4. DELETE Operations

### Basic DELETE

```
*- Delete a single row*
DELETE FROM users
WHERE id = 5;
```

### DELETE Multiple Rows

```
*- Delete multiple rows*
DELETE FROM users
WHERE status = 'inactive';
```

### DELETE with JOIN

```
*- Delete based on data in another table*
DELETE o
FROM orders o
JOIN users u ON o.user_id = u.id
WHERE u.status = 'suspended';
```

### Safe DELETE (Batch Processing)

```
*- Batch processing for large deletions (prevents locking)*
SET @batch_size = 1000;
DELETE FROM large_table
WHERE created_at < '2023-01-01'
LIMIT @batch_size;
```

These examples cover the core CRUD operations in MySQL, allowing you to create, read, update, and delete data in your database efficiently.

---

## ✈️ Happy Coding!

---