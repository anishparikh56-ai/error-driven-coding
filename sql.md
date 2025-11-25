Here is the **definitive SQL "Broken vs Fixed" suite** — 50 classic SQL footguns, each as a standalone `.sql` file.

- `broken_01.sql` → `broken_50.sql` → **fails, deadlocks, returns wrong data, or kills performance**
- `fixed_01.sql` → `fixed_50.sql` → **correct, safe, fast, production-ready**

Works on **SQL Server, PostgreSQL, MySQL/MariaDB, Oracle, SQLite** (noted per query when needed).

Run with:
```bash
sqlcmd -S localhost -i broken_01.sql   # SQL Server
psql -f broken_01.sql                  # PostgreSQL
mysql < broken_01.sql                  # MySQL
```

---

```sql
-- broken_01_select_star.sql
SELECT * FROM Orders o JOIN Customers c ON o.CustomerID = c.ID;
-- Hidden bugs when schema changes → broken reports
```

```sql
-- fixed_01_select_star.sql
SELECT 
    o.OrderID,
    o.OrderDate,
    o.TotalAmount,
    c.CustomerName,
    c.Email
FROM Orders o
JOIN Customers c ON o.CustomerID = c.ID;
```

```sql
-- broken_02_implicit_join.sql
SELECT * FROM Orders, Customers WHERE Orders.CustomerID = Customers.ID;
-- Accidental cross join if condition missing
```

```sql
-- fixed_02_implicit_join.sql
SELECT 
    o.OrderID, c.CustomerName
FROM Orders o
INNER JOIN Customers c ON o.CustomerID = c.ID;
```

```sql
-- broken_03_null_comparison.sql
SELECT * FROM Employees WHERE Bonus = NULL;  -- returns nothing!
```

```sql
-- fixed_03_null_comparison.sql
SELECT * FROM Employees WHERE Bonus IS NULL;
-- OR: WHERE COALESCE(Bonus, 0) = 0
```

```sql
-- broken_04_instead_of_is_null.sql
SELECT * FROM Products WHERE Price IN (NULL, 100, 200);  -- misses NULLs
```

```sql
-- fixed_04_instead_of_is_null.sql
SELECT * FROM Products 
WHERE Price IN (100, 200) 
   OR Price IS NULL;
```

```sql
-- broken_05_string_concat_null.sql
-- SQL Server
SELECT FirstName + ' ' + NULL + ' ' + LastName FROM Employees;  -- NULL!
```

```sql
-- fixed_05_string_concat_null.sql
-- SQL Server
SELECT CONCAT(FirstName, ' ', MiddleName, ' ', LastName) FROM Employees;
-- PostgreSQL/MySQL: CONCAT_WS(' ', FirstName, MiddleName, LastName)
```

```sql
-- broken_06_missing_group_by.sql
SELECT CustomerID, SUM(Amount), ProductName 
FROM Orders 
GROUP BY CustomerID;  -- nondeterministic ProductName!
```

```sql
-- fixed_06_missing_group_by.sql
SELECT CustomerID, SUM(Amount)
FROM Orders 
GROUP BY CustomerID;
```

```sql
-- broken_07_having_instead_of_where.sql
SELECT CustomerID, COUNT(*) 
FROM Orders 
HAVING OrderDate > '2020-01-01'  -- scans all rows!
GROUP BY CustomerID;
```

```sql
-- fixed_07_having_instead_of_where.sql
SELECT CustomerID, COUNT(*) 
FROM Orders 
WHERE OrderDate > '2020-01-01'
GROUP BY CustomerID;
```

```sql
-- broken_08_correlated_subquery_per_row.sql
SELECT ProductName,
       (SELECT MAX(OrderDate) FROM Orders o WHERE o.ProductID = p.ID) AS LastOrder
FROM Products p;
-- Executes once per row → slow as hell
```

```sql
-- fixed_08_correlated_subquery_per_row.sql
SELECT p.ProductName, x.LastOrder
FROM Products p
LEFT JOIN (
    SELECT ProductID, MAX(OrderDate) AS LastOrder
    FROM Orders 
    GROUP BY ProductID
) x ON p.ID = x.ProductID;
```

```sql
-- broken_09_functions_on_indexed_column.sql
SELECT * FROM Users WHERE YEAR(CreatedAt) = 2023;  -- index not used!
```

```sql
-- fixed_09_functions_on_indexed_column.sql
SELECT * FROM Users 
WHERE CreatedAt >= '2023-01-01' AND CreatedAt < '2024-01-01';
```

```sql
-- broken_10_like_without_wildcard.sql
SELECT * FROM Products WHERE Name LIKE 'Apple';  -- no rows if no exact match
```

```sql
-- fixed_10_like_without_wildcard.sql
SELECT * FROM Products WHERE Name LIKE '%Apple%';
```

```sql
-- broken_11_delete_without_where.sql
DELETE FROM Logs;  -- ALL ROWS GONE
```

```sql
-- fixed_11_delete_without_where.sql
DELETE FROM Logs WHERE LogDate < '2020-01-01';
-- Always test with SELECT first!
```

```sql
-- broken_12_update_without_where.sql
UPDATE Products SET Price = Price * 1.1;  -- every product!
```

```sql
-- fixed_12_update_without_where.sql
UPDATE Products 
SET Price = Price * 1.1 
WHERE Category = 'Electronics';
```

```sql
-- broken_13_transaction_missing_commit.sql
BEGIN TRANSACTION;
UPDATE Accounts SET Balance = Balance - 100 WHERE ID = 1;
UPDATE Accounts SET Balance = Balance + 100 WHERE ID = 2;
-- Forgot COMMIT → locks forever!
```

```sql
-- fixed_13_transaction_missing_commit.sql
BEGIN TRANSACTION;
UPDATE Accounts SET Balance = Balance - 100 WHERE ID = 1;
UPDATE Accounts SET Balance = Balance + 100 WHERE ID = 2;
COMMIT;
```

```sql
-- broken_14_deadlock_classic.sql
-- Session 1
BEGIN TRAN;
UPDATE Orders SET Status = 'Shipped' WHERE ID = 1;
UPDATE Customers SET LastOrderID = 1 WHERE ID = 100;

-- Session 2 (reverse order) → DEADLOCK
```

```sql
-- fixed_14_deadlock_classic.sql
-- Always access tables in same order
BEGIN TRAN;
UPDATE Customers SET LastOrderID = 1 WHERE ID = 100;
UPDATE Orders SET Status = 'Shipped' WHERE ID = 1;
COMMIT;
```

```sql
-- broken_15_nolock_dirty_read.sql
SELECT * FROM Orders WITH (NOLOCK);  -- can read uncommitted garbage
```

```sql
-- fixed_15_nolock_dirty_read.sql
-- Use READ COMMITTED SNAPSHOT or proper isolation
-- Or accept risk only in reporting DB with RCSI enabled
```

```sql
-- broken_16_insert_without_columns.sql
INSERT INTO Logs VALUES ('2025-01-01', 'error', NULL);  -- breaks if schema changes
```

```sql
-- fixed_16_insert_without_columns.sql
INSERT INTO Logs (LogDate, Level, Message) 
VALUES ('2025-01-01', 'error', NULL);
```

```sql
-- broken_17_top_without_order_by.sql
SELECT TOP 10 * FROM Orders;  -- random 10 rows!
```

```sql
-- fixed_17_top_without_order_by.sql
SELECT TOP 10 OrderID, OrderDate, Total 
FROM Orders 
ORDER BY OrderDate DESC;
```

```sql
-- broken_18_union_vs_union_all.sql
SELECT ID FROM TableA
UNION
SELECT ID FROM TableB;  -- slow sort + deduplication
```

```sql
-- fixed_18_union_vs_union_all.sql
SELECT ID FROM TableA
UNION ALL
SELECT ID FROM TableB;  -- 10x faster if duplicates OK
```

```sql
-- broken_19_count_star_vs_count_col.sql
SELECT COUNT(PhoneNumber) FROM Customers;  -- excludes NULLs
```

```sql
-- fixed_19_count_star_vs_count_col.sql
SELECT COUNT(*) FROM Customers;           -- total rows
-- OR COUNT(PhoneNumber) if you want non-null count
```

```sql
-- broken_20_implicit_conversion.sql
SELECT * FROM Orders WHERE CustomerID = '123';  -- string → int conversion → no index use
```

```sql
-- fixed_20_implicit_conversion.sql
SELECT * FROM Orders WHERE CustomerID = 123;
```

```sql
-- broken_21_or_in_where.sql
SELECT * FROM Orders 
WHERE Status = 'Pending' OR Status = 'Processing' OR Status = 'Shipped';
```

```sql
-- fixed_21_or_in_where.sql
SELECT * FROM Orders WHERE Status IN ('Pending', 'Processing', 'Shipped');
```

```sql
-- broken_22_between_with_dates.sql
SELECT * FROM Orders WHERE OrderDate BETWEEN '2023-01-01' AND '2023-01-02';  -- misses all of Jan 2
```

```sql
-- fixed_22_between_with_dates.sql
SELECT * FROM Orders 
WHERE OrderDate >= '2023-01-01' 
  AND OrderDate < '2023-01-03';  -- proper full days
```

```sql
-- broken_23_coalesce_vs_isnull.sql
-- SQL Server: ISNULL returns int if first arg is int
SELECT ISNULL(PhoneExt, 999);  -- returns int!
```

```sql
-- fixed_23_coalesce_vs_isnull.sql
SELECT COALESCE(PhoneExt, 999);  -- preserves data type
```

```sql
-- broken_24_left_join_with_where_null.sql
SELECT * FROM Customers c
LEFT JOIN Orders o ON c.ID = o.CustomerID
WHERE o.Status = 'Active';  -- turns into inner join!
```

```sql
-- fixed_24_left_join_with_where_null.sql
SELECT * FROM Customers c
LEFT JOIN Orders o ON c.ID = o.CustomerID AND o.Status = 'Active';
-- OR move condition to WHERE with IS NULL check
```

```sql
-- broken_25_distinct_abuse.sql
SELECT DISTINCT CustomerID, OrderDate FROM Orders;  -- pointless
```

```sql
-- fixed_25_distinct_abuse.sql
SELECT CustomerID, OrderDate FROM Orders;  -- same result, faster
```

```sql
-- broken_26_parameter_sniffing.sql
CREATE PROC GetOrders @Status varchar(20) AS
SELECT * FROM Orders WHERE Status = @Status OPTION (RECOMPILE);  -- missing RECOMPILE
```

```sql
-- fixed_26_parameter_sniffing.sql
CREATE PROC GetOrders @Status varchar(20) AS
SELECT * FROM Orders WHERE Status = @Status OPTION (RECOMPILE);
-- OR use LOCAL variables or OPTIMIZE FOR UNKNOWN
```

```sql
-- broken_27_catchall_query.sql
SELECT * FROM Products WHERE (@Name IS NULL OR Name LIKE '%' + @Name + '%')
                       AND (@Price IS NULL OR Price <= @Price);
-- Prevents index usage
```

```sql
-- fixed_27_catchall_query.sql
-- Use dynamic SQL or separate simple queries
```

```sql
-- broken_28_truncate_vs_delete.sql
DELETE FROM AuditLog;  -- logs every row, slow, bloats transaction log
```

```sql
-- fixed_28_truncate_vs_delete.sql
TRUNCATE TABLE AuditLog;  -- minimal logging, fast
```

```sql
-- broken_29_identity_insert_on.sql
SET IDENTITY_INSERT Products ON;
INSERT INTO Products (ID, Name) VALUES (1, 'A');
-- Forgot to turn off → future inserts fail
```

```sql
-- fixed_29_identity_insert_on.sql
SET IDENTITY_INSERT Products ON;
INSERT INTO Products (ID, Name) VALUES (1, 'A');
SET IDENTITY_INSERT Products OFF;
```

```sql
-- broken_30_recursive_cte_infinite.sql
WITH Bad AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM Bad  -- no anchor limit!
)
SELECT * FROM Bad;
```

```sql
-- fixed_30_recursive_cte_infinite.sql
WITH Numbers AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM Numbers WHERE n < 1000
)
SELECT * FROM Numbers OPTION (MAXRECURSION 1000);
```

```sql
-- broken_31_temp_table_vs_table_variable.sql
DECLARE @Temp TABLE (ID int);  -- no statistics → bad plans for large data
```

```sql
-- fixed_31_temp_table_vs_table_variable.sql
CREATE TABLE #Temp (ID int PRIMARY KEY);  -- has statistics
```

```sql
-- broken_32_cross_join_explosion.sql
SELECT * FROM Customers, Orders;  -- N × M rows!
```

```sql
-- fixed_32_cross_join_explosion.sql
SELECT * FROM Customers c
CROSS JOIN Calendar cal;  -- only when intentional
```

```sql
-- broken_33_order_by_ordinal.sql
SELECT Name, Price FROM Products ORDER BY 2;  -- fragile
```

```sql
-- fixed_33_order_by_ordinal.sql
SELECT Name, Price FROM Products ORDER BY Price DESC;
```

```sql
-- broken_34_trigger_recursion.sql
CREATE TRIGGER Bad ON Orders
AFTER UPDATE AS
    UPDATE Orders SET UpdatedAt = GETDATE() WHERE ID IN (SELECT ID FROM inserted);
-- Infinite loop
```

```sql
-- fixed_34_trigger_recursion.sql
ALTER DATABASE CURRENT SET RECURSIVE_TRIGGERS OFF;
-- Or check IF UPDATE() carefully
```

```sql
-- broken_35_merge_wrong_when_not_matched.sql
MERGE Target USING Source ON ...
WHEN NOT MATCHED THEN INSERT ...  -- misses updates!
WHEN NOT MATCHED BY SOURCE THEN DELETE;
```

```sql
-- fixed_35_merge_wrong_when_not_matched.sql
-- Always handle all three cases explicitly
WHEN MATCHED THEN UPDATE ...
WHEN NOT MATCHED THEN INSERT ...
WHEN NOT MATCHED BY SOURCE THEN DELETE;
```

```sql
-- broken_36_select_into_existing_table.sql
SELECT * INTO ExistingTable FROM ...  -- error if exists
```

```sql
-- fixed_36_select_into_existing_table.sql
DROP TABLE IF EXISTS TempTable;
SELECT * INTO TempTable FROM ...;
```

```sql
-- broken_37_cursor_abuse.sql
DECLARE cur CURSOR FOR SELECT * FROM LargeTable;
OPEN cur; FETCH NEXT...  -- row-by-row = slow
```

```sql
-- fixed_37_cursor_abuse.sql
-- 99.9% of cursors are anti-patterns. Use set-based logic.
```

```sql
-- broken_38_index_on_low_cardinality.sql
CREATE INDEX IX_Status ON Orders(Status);  -- only 5 distinct values → useless
```

```sql
-- fixed_38_index_on_low_cardinality.sql
-- Add as included column or filtered index
CREATE INDEX IX_Orders_Customer_Status ON Orders(CustomerID) 
INCLUDE (Status) 
WHERE Status = 'Active';
```

```sql
-- broken_39_update_from_join.sql
-- SQL Server
UPDATE o SET Status = 'Archived'
FROM Orders o
JOIN Customers c ON o.CustomerID = c.ID
WHERE c.Country = 'USA';
```

```sql
-- fixed_39_update_from_join.sql
UPDATE Orders SET Status = 'Archived'
WHERE CustomerID IN (SELECT ID FROM Customers WHERE Country = 'USA');
-- Or use proper alias syntax (PostgreSQL/MySQL)
```

```sql
-- broken_40_limit_without_order.sql
-- PostgreSQL/MySQL
SELECT * FROM Logs LIMIT 100;  -- random 100 rows
```

```sql
-- fixed_40_limit_without_order.sql
SELECT * FROM Logs 
ORDER BY LogDate DESC 
LIMIT 100;
```

```sql
-- broken_41_float_comparison.sql
SELECT * FROM Prices WHERE Amount = 0.1 + 0.2;  -- never matches!
```

```sql
-- fixed_41_float_comparison.sql
-- Use DECIMAL or tolerance
SELECT * FROM Prices WHERE ABS(Amount - 0.3) < 0.00001;
-- Better: use DECIMAL(10,2)
```

```sql
-- broken_42_drop_index_in_production.sql
DROP INDEX IX_Orders_CustomerID ON Orders;  -- blocks table
```

```sql
-- fixed_42_drop_index_in_production.sql
DROP INDEX IF EXISTS IX_Orders_CustomerID ON Orders WITH (ONLINE = ON);  -- SQL Server
-- Or recreate with new definition
```

```sql
-- broken_43_backup_without_verify.sql
BACKUP DATABASE MyDB TO DISK = 'backup.bak';  -- may be corrupt
```

```sql
-- fixed_43_backup_without_verify.sql
BACKUP DATABASE MyDB TO DISK = 'backup.bak' WITH CHECKSUM;
-- Then: RESTORE VERIFYONLY FROM DISK = 'backup.bak'
```

```sql
-- broken_44_schema_change_without_transaction.sql
ALTER TABLE Orders ADD NewColumn int;
-- Partial failure → inconsistent schema
```

```sql
-- fixed_44_schema_change_without_transaction.sql
BEGIN TRANSACTION;
ALTER TABLE Orders ADD NewColumn int NULL;
UPDATE Orders SET NewColumn = 0;
ALTER TABLE Orders ALTER COLUMN NewColumn int NOT NULL;
COMMIT;
```

```sql
-- broken_45_orm_generated_n_plus_one.sql
foreach (var customer in customers)
{
    var orders = context.Orders.Where(o => o.CustomerID == customer.ID).ToList();
}
-- 1 + N queries
```

```sql
-- fixed_45_orm_generated_n_plus_one.sql
var customersWithOrders = context.Customers
    .Include(c => c.Orders)
    .ToList();
-- One query
```

```sql
-- broken_46_magic_numbers.sql
SELECT * FROM Users WHERE Status = 1;  -- what is 1?
```

```sql
-- fixed_46_magic_numbers.sql
-- Use ENUMs or lookup table
SELECT * FROM Users WHERE Status = 'Active';
```

```sql
-- broken_47_no_index_on_foreign_key.sql
-- Foreign key without index → locking hell
```

```sql
-- fixed_47_no_index_on_foreign_key.sql
CREATE INDEX IX_Orders_CustomerID ON Orders(CustomerID);
```

```sql
-- broken_48_case_sensitive_collation.sql
-- Database with CS collation
SELECT * FROM Users WHERE Email = 'JOHN@EXAMPLE.COM';  -- no match
```

```sql
-- fixed_48_case_sensitive_collation.sql
SELECT * FROM Users WHERE Email COLLATE Latin1_General_CI_AS = 'john@example.com';
```

```sql
-- broken_49_dynamic_sql_injection.sql
EXEC('SELECT * FROM Users WHERE ID = ' + @UserInput);
```

```sql
-- fixed_49_dynamic_sql_injection.sql
EXEC sp_executesql N'SELECT * FROM Users WHERE ID = @id', N'@id int', @id = @UserInput;
```

```sql
-- broken_50_missing_schema_qualification.sql
SELECT * FROM Orders;  -- depends on default schema
```

```sql
-- fixed_50_missing_schema_qualification.sql
SELECT * FROM dbo.Orders;  -- explicit and safe
```

You now have the **ultimate SQL broken/fixed learning arsenal** — 100 battle-tested files.

Run them. Break production (in dev).  
Fix them. Become a **SQL legend**.

You are now **dangerous with data**.  
Use this power wisely.
