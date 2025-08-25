# SQL Server - Hướng dẫn đầy đủ cho công việc

> **Tài liệu này chứa 100% kiến thức SQL Server BẮT BUỘC khi đi làm**  
> Bao gồm code mẫu thực tế và best practices từ kinh nghiệm doanh nghiệp

## 📋 Mục lục

- [1. Tạo Database và Tables](#1-tạo-database-và-tables)
- [2. Insert dữ liệu mẫu](#2-insert-dữ-liệu-mẫu)
- [3. Câu lệnh SELECT cơ bản (100% BẮT BUỘC)](#3-câu-lệnh-select-cơ-bản-100-bắt-buộc)
- [4. GROUP BY và Aggregate Functions](#4-group-by-và-aggregate-functions)
- [5. JOINS (CỰC KỲ QUAN TRỌNG)](#5-joins-cực-kỳ-quan-trọng)
- [6. Subqueries và CTE](#6-subqueries-và-cte)
- [7. CASE WHEN (Logic điều kiện)](#7-case-when-logic-điều-kiện)
- [8. T-SQL Variables và Control Flow](#8-t-sql-variables-và-control-flow)
- [9. Error Handling (TRY/CATCH)](#9-error-handling-trycatch)
- [10. Stored Procedures (BẮT BUỘC BIẾT)](#10-stored-procedures-bắt-buộc-biết)
- [11. Functions (User-Defined)](#11-functions-user-defined)
- [12. Views](#12-views)
- [13. String Functions (HAY DÙNG)](#13-string-functions-hay-dùng)
- [14. Date Functions (CỰC KỲ QUAN TRỌNG)](#14-date-functions-cực-kỳ-quan-trọng)
- [15. NULL Handling](#15-null-handling)
- [16. Transactions (QUAN TRỌNG)](#16-transactions-quan-trọng)
- [17. Indexes (Cải thiện Performance)](#17-indexes-cải-thiện-performance)
- [18. Window Functions (NÂNG CAO)](#18-window-functions-nâng-cao)
- [19. PIVOT và UNPIVOT](#19-pivot-và-unpivot)
- [20. Dynamic SQL](#20-dynamic-sql)
- [21. Backup và Restore](#21-backup-và-restore)
- [22. Permissions và Security](#22-permissions-và-security)
- [23. Performance Monitoring](#23-performance-monitoring)
- [24. System Queries hữu ích](#24-system-queries-hữu-ích)
- [25. Common Table Expressions (CTE) - Nâng cao](#25-common-table-expressions-cte---nâng-cao)
- [26. XML Functions](#26-xml-functions)
- [27. JSON Functions](#27-json-functions)
- [28. Temporary Tables và Table Variables](#28-temporary-tables-và-table-variables)
- [29. Cursors](#29-cursors)
- [30. MERGE Statement (UPSERT)](#30-merge-statement-upsert)
- [31. Common Patterns và Best Practices](#31-common-patterns-và-best-practices)
- [32. Performance Tips & Tricks](#32-performance-tips--tricks)
- [33. Error Handling Patterns](#33-error-handling-patterns)
- [34. Maintenance Commands](#34-maintenance-commands)
- [35. Common Business Scenarios](#35-common-business-scenarios)

---

## 1. Tạo Database và Tables

```sql
-- Tạo database
CREATE DATABASE CompanyDB;
GO

USE CompanyDB;
GO

-- Tạo bảng Departments
CREATE TABLE Departments (
    DepartmentID INT IDENTITY(1,1) PRIMARY KEY,
    DepartmentName NVARCHAR(100) NOT NULL,
    Budget DECIMAL(15,2) NULL,
    CreatedDate DATETIME DEFAULT GETDATE()
);

-- Tạo bảng Employees
CREATE TABLE Employees (
    EmployeeID INT IDENTITY(1,1) PRIMARY KEY,
    FirstName NVARCHAR(50) NOT NULL,
    LastName NVARCHAR(50) NOT NULL,
    Email NVARCHAR(100) UNIQUE,
    Phone NVARCHAR(20),
    HireDate DATE NOT NULL,
    Salary DECIMAL(10,2),
    DepartmentID INT,
    IsActive BIT DEFAULT 1,
    FOREIGN KEY (DepartmentID) REFERENCES Departments(DepartmentID)
);

-- Tạo bảng Projects
CREATE TABLE Projects (
    ProjectID INT IDENTITY(1,1) PRIMARY KEY,
    ProjectName NVARCHAR(200) NOT NULL,
    StartDate DATE,
    EndDate DATE,
    Budget DECIMAL(15,2),
    Status NVARCHAR(20) DEFAULT 'Active'
);

-- Tạo bảng EmployeeProjects (Many-to-Many)
CREATE TABLE EmployeeProjects (
    EmployeeID INT,
    ProjectID INT,
    Role NVARCHAR(50),
    AssignedDate DATE DEFAULT GETDATE(),
    PRIMARY KEY (EmployeeID, ProjectID),
    FOREIGN KEY (EmployeeID) REFERENCES Employees(EmployeeID),
    FOREIGN KEY (ProjectID) REFERENCES Projects(ProjectID)
);
```

## 2. Insert dữ liệu mẫu

```sql
-- Insert Departments
INSERT INTO Departments (DepartmentName, Budget) VALUES 
('IT', 500000.00),
('HR', 200000.00),
('Finance', 300000.00),
('Marketing', 250000.00);

-- Insert Employees
INSERT INTO Employees (FirstName, LastName, Email, Phone, HireDate, Salary, DepartmentID) VALUES 
('Nguyen', 'Van A', 'nguyenvana@company.com', '0901234567', '2020-01-15', 15000000, 1),
('Tran', 'Thi B', 'tranthib@company.com', '0901234568', '2020-03-20', 12000000, 2),
('Le', 'Van C', 'levanc@company.com', '0901234569', '2019-05-10', 18000000, 1),
('Pham', 'Thi D', 'phamthid@company.com', '0901234570', '2021-07-01', 14000000, 3),
('Hoang', 'Van E', 'hoangvane@company.com', '0901234571', '2022-01-15', 13000000, 4);

-- Insert Projects
INSERT INTO Projects (ProjectName, StartDate, EndDate, Budget, Status) VALUES 
('Website Redesign', '2024-01-01', '2024-06-30', 100000000, 'Active'),
('ERP Implementation', '2024-02-01', '2024-12-31', 500000000, 'Active'),
('Marketing Campaign Q1', '2024-01-01', '2024-03-31', 50000000, 'Completed');

-- Insert EmployeeProjects
INSERT INTO EmployeeProjects (EmployeeID, ProjectID, Role) VALUES 
(1, 1, 'Developer'),
(3, 1, 'Team Lead'),
(1, 2, 'Developer'),
(4, 2, 'Business Analyst'),
(5, 3, 'Marketing Specialist');
```

## 3. Câu lệnh SELECT cơ bản (100% BẮT BUỘC)

```sql
-- SELECT đơn giản
SELECT * FROM Employees;
SELECT FirstName, LastName, Salary FROM Employees;

-- WHERE với nhiều điều kiện
SELECT * FROM Employees 
WHERE Salary > 14000000 AND IsActive = 1;

-- LIKE pattern matching
SELECT * FROM Employees 
WHERE FirstName LIKE 'N%' OR Email LIKE '%@company.com';

-- IN và NOT IN
SELECT * FROM Employees 
WHERE DepartmentID IN (1, 2);

-- BETWEEN
SELECT * FROM Employees 
WHERE HireDate BETWEEN '2020-01-01' AND '2021-12-31';

-- IS NULL và IS NOT NULL
SELECT * FROM Employees 
WHERE Phone IS NOT NULL;

-- ORDER BY
SELECT * FROM Employees 
ORDER BY Salary DESC, LastName ASC;

-- TOP và PERCENT
SELECT TOP 3 * FROM Employees ORDER BY Salary DESC;
SELECT TOP 50 PERCENT * FROM Employees ORDER BY HireDate;
```

## 4. GROUP BY và Aggregate Functions

```sql
-- Đếm số nhân viên theo phòng ban
SELECT 
    d.DepartmentName,
    COUNT(e.EmployeeID) as EmployeeCount,
    AVG(e.Salary) as AvgSalary,
    MIN(e.Salary) as MinSalary,
    MAX(e.Salary) as MaxSalary,
    SUM(e.Salary) as TotalSalary
FROM Departments d
LEFT JOIN Employees e ON d.DepartmentID = e.DepartmentID
GROUP BY d.DepartmentID, d.DepartmentName;

-- GROUP BY với HAVING
SELECT 
    DepartmentID,
    COUNT(*) as EmployeeCount,
    AVG(Salary) as AvgSalary
FROM Employees 
GROUP BY DepartmentID
HAVING COUNT(*) > 1 AND AVG(Salary) > 13000000;
```

## 5. JOINS (CỰC KỲ QUAN TRỌNG)

```sql
-- INNER JOIN
SELECT 
    e.FirstName,
    e.LastName,
    d.DepartmentName,
    e.Salary
FROM Employees e
INNER JOIN Departments d ON e.DepartmentID = d.DepartmentID;

-- LEFT JOIN
SELECT 
    d.DepartmentName,
    e.FirstName,
    e.LastName
FROM Departments d
LEFT JOIN Employees e ON d.DepartmentID = e.DepartmentID;

-- RIGHT JOIN
SELECT 
    e.FirstName,
    e.LastName,
    d.DepartmentName
FROM Departments d
RIGHT JOIN Employees e ON d.DepartmentID = e.DepartmentID;

-- FULL OUTER JOIN
SELECT 
    ISNULL(d.DepartmentName, 'No Department') as Department,
    ISNULL(e.FirstName + ' ' + e.LastName, 'No Employee') as EmployeeName
FROM Departments d
FULL OUTER JOIN Employees e ON d.DepartmentID = e.DepartmentID;

-- Multiple JOINS
SELECT 
    e.FirstName + ' ' + e.LastName as EmployeeName,
    d.DepartmentName,
    p.ProjectName,
    ep.Role
FROM Employees e
INNER JOIN Departments d ON e.DepartmentID = d.DepartmentID
INNER JOIN EmployeeProjects ep ON e.EmployeeID = ep.EmployeeID
INNER JOIN Projects p ON ep.ProjectID = p.ProjectID;
```

## 6. Subqueries và CTE

```sql
-- Subquery trong WHERE
SELECT * FROM Employees 
WHERE Salary > (SELECT AVG(Salary) FROM Employees);

-- Subquery trong SELECT
SELECT 
    FirstName,
    LastName,
    Salary,
    (SELECT AVG(Salary) FROM Employees) as AvgSalary,
    Salary - (SELECT AVG(Salary) FROM Employees) as SalaryDifference
FROM Employees;

-- EXISTS
SELECT * FROM Employees e
WHERE EXISTS (
    SELECT 1 FROM EmployeeProjects ep 
    WHERE ep.EmployeeID = e.EmployeeID
);

-- CTE (Common Table Expression) - Cực kỳ hữu ích
WITH HighSalaryEmployees AS (
    SELECT 
        EmployeeID,
        FirstName,
        LastName,
        Salary,
        DepartmentID
    FROM Employees 
    WHERE Salary > 14000000
),
DepartmentStats AS (
    SELECT 
        DepartmentID,
        COUNT(*) as EmpCount,
        AVG(Salary) as AvgSalary
    FROM HighSalaryEmployees
    GROUP BY DepartmentID
)
SELECT 
    hse.FirstName,
    hse.LastName,
    hse.Salary,
    ds.AvgSalary,
    d.DepartmentName
FROM HighSalaryEmployees hse
INNER JOIN DepartmentStats ds ON hse.DepartmentID = ds.DepartmentID
INNER JOIN Departments d ON hse.DepartmentID = d.DepartmentID;
```

## 7. CASE WHEN (Logic điều kiện)

```sql
SELECT 
    FirstName,
    LastName,
    Salary,
    CASE 
        WHEN Salary >= 18000000 THEN 'Senior'
        WHEN Salary >= 15000000 THEN 'Middle'
        WHEN Salary >= 12000000 THEN 'Junior'
        ELSE 'Intern'
    END as Level,
    CASE 
        WHEN DATEDIFF(YEAR, HireDate, GETDATE()) >= 3 THEN 'Experienced'
        ELSE 'New'
    END as Experience
FROM Employees;

-- CASE trong GROUP BY
SELECT 
    CASE 
        WHEN Salary >= 15000000 THEN 'High Salary'
        ELSE 'Normal Salary'
    END as SalaryGroup,
    COUNT(*) as EmployeeCount,
    AVG(Salary) as AvgSalary
FROM Employees
GROUP BY 
    CASE 
        WHEN Salary >= 15000000 THEN 'High Salary'
        ELSE 'Normal Salary'
    END;
```

## 8. T-SQL Variables và Control Flow

```sql
-- Variables
DECLARE @MinSalary DECIMAL(10,2) = 12000000;
DECLARE @DeptName NVARCHAR(100);
DECLARE @EmployeeCount INT;

-- SET vs SELECT
SET @DeptName = 'IT';
SELECT @EmployeeCount = COUNT(*) FROM Employees WHERE DepartmentID = 1;

PRINT 'Department: ' + @DeptName;
PRINT 'Employee Count: ' + CAST(@EmployeeCount AS NVARCHAR(10));

-- IF/ELSE
IF @EmployeeCount > 2
BEGIN
    PRINT 'Large department';
    SELECT * FROM Employees WHERE DepartmentID = 1;
END
ELSE
BEGIN
    PRINT 'Small department';
END

-- WHILE Loop
DECLARE @Counter INT = 1;
WHILE @Counter <= 5
BEGIN
    PRINT 'Counter: ' + CAST(@Counter AS NVARCHAR(10));
    SET @Counter = @Counter + 1;
END
```

## 9. Error Handling (TRY/CATCH)

```sql
BEGIN TRY
    -- Code có thể gây lỗi
    INSERT INTO Employees (FirstName, LastName, Email, HireDate, Salary, DepartmentID) 
    VALUES ('Test', 'User', 'duplicate@company.com', GETDATE(), 15000000, 1);
    
    PRINT 'Insert successful';
END TRY
BEGIN CATCH
    PRINT 'Error occurred:';
    PRINT 'Error Number: ' + CAST(ERROR_NUMBER() AS NVARCHAR(10));
    PRINT 'Error Message: ' + ERROR_MESSAGE();
    PRINT 'Error Line: ' + CAST(ERROR_LINE() AS NVARCHAR(10));
END CATCH
```

## 10. Stored Procedures (BẮT BUỘC BIẾT)

```sql
-- Stored Procedure đơn giản
CREATE PROCEDURE GetEmployeesByDepartment
    @DepartmentID INT
AS
BEGIN
    SELECT 
        FirstName,
        LastName,
        Email,
        Salary
    FROM Employees 
    WHERE DepartmentID = @DepartmentID AND IsActive = 1
    ORDER BY LastName;
END
GO

-- Gọi Stored Procedure
EXEC GetEmployeesByDepartment @DepartmentID = 1;

-- Stored Procedure với Output Parameter
CREATE PROCEDURE GetEmployeeStats
    @DepartmentID INT,
    @TotalEmployees INT OUTPUT,
    @AvgSalary DECIMAL(10,2) OUTPUT
AS
BEGIN
    SELECT 
        @TotalEmployees = COUNT(*),
        @AvgSalary = AVG(Salary)
    FROM Employees 
    WHERE DepartmentID = @DepartmentID AND IsActive = 1;
END
GO

-- Sử dụng Output Parameter
DECLARE @Total INT, @Avg DECIMAL(10,2);
EXEC GetEmployeeStats 
    @DepartmentID = 1,
    @TotalEmployees = @Total OUTPUT,
    @AvgSalary = @Avg OUTPUT;

PRINT 'Total Employees: ' + CAST(@Total AS NVARCHAR(10));
PRINT 'Average Salary: ' + CAST(@Avg AS NVARCHAR(20));
```

## 11. Functions (User-Defined)

```sql
-- Scalar Function
CREATE FUNCTION GetEmployeeFullName(@EmployeeID INT)
RETURNS NVARCHAR(101)
AS
BEGIN
    DECLARE @FullName NVARCHAR(101);
    
    SELECT @FullName = FirstName + ' ' + LastName
    FROM Employees 
    WHERE EmployeeID = @EmployeeID;
    
    RETURN ISNULL(@FullName, 'Unknown');
END
GO

-- Sử dụng Function
SELECT dbo.GetEmployeeFullName(1) as FullName;

-- Table-Valued Function
CREATE FUNCTION GetEmployeesByDepartmentTVF(@DepartmentID INT)
RETURNS TABLE
AS
RETURN (
    SELECT 
        EmployeeID,
        FirstName + ' ' + LastName as FullName,
        Email,
        Salary,
        HireDate
    FROM Employees 
    WHERE DepartmentID = @DepartmentID AND IsActive = 1
);
GO

-- Sử dụng Table-Valued Function
SELECT * FROM dbo.GetEmployeesByDepartmentTVF(1);
```

## 12. Views

```sql
-- Tạo View
CREATE VIEW vw_EmployeeDepartmentInfo AS
SELECT 
    e.EmployeeID,
    e.FirstName + ' ' + e.LastName as FullName,
    e.Email,
    e.Salary,
    e.HireDate,
    d.DepartmentName,
    DATEDIFF(YEAR, e.HireDate, GETDATE()) as YearsOfService,
    CASE 
        WHEN e.Salary >= 18000000 THEN 'Senior'
        WHEN e.Salary >= 15000000 THEN 'Middle'
        ELSE 'Junior'
    END as Level
FROM Employees e
INNER JOIN Departments d ON e.DepartmentID = d.DepartmentID
WHERE e.IsActive = 1;
GO

-- Sử dụng View
SELECT * FROM vw_EmployeeDepartmentInfo;
SELECT * FROM vw_EmployeeDepartmentInfo WHERE Level = 'Senior';
```

## 13. String Functions (HAY DÙNG)

```sql
SELECT 
    FirstName,
    LastName,
    -- Nối chuỗi
    FirstName + ' ' + LastName as FullName,
    CONCAT(FirstName, ' ', LastName) as FullName2,
    
    -- Độ dài chuỗi
    LEN(FirstName) as FirstNameLength,
    
    -- Cắt chuỗi
    SUBSTRING(Email, 1, CHARINDEX('@', Email) - 1) as Username,
    RIGHT(Email, LEN(Email) - CHARINDEX('@', Email)) as Domain,
    LEFT(Phone, 3) as AreaCode,
    
    -- Thay thế
    REPLACE(Phone, '090', '084') as UpdatedPhone,
    
    -- Uppercase/Lowercase
    UPPER(FirstName) as FirstNameUpper,
    LOWER(LastName) as LastNameLower,
    
    -- Trim spaces
    LTRIM(RTRIM(FirstName)) as TrimmedName,
    
    -- Reverse
    REVERSE(FirstName) as ReversedName
FROM Employees;
```

## 14. Date Functions (CỰC KỲ QUAN TRỌNG)

```sql
SELECT 
    FirstName,
    LastName,
    HireDate,
    
    -- Current date/time
    GETDATE() as CurrentDateTime,
    GETUTCDATE() as CurrentUTCDateTime,
    
    -- Date parts
    YEAR(HireDate) as HireYear,
    MONTH(HireDate) as HireMonth,
    DAY(HireDate) as HireDay,
    DATENAME(WEEKDAY, HireDate) as HireDayOfWeek,
    DATEPART(QUARTER, HireDate) as HireQuarter,
    
    -- Date calculations
    DATEDIFF(YEAR, HireDate, GETDATE()) as YearsOfService,
    DATEDIFF(MONTH, HireDate, GETDATE()) as MonthsOfService,
    DATEDIFF(DAY, HireDate, GETDATE()) as DaysOfService,
    
    -- Add/Subtract dates
    DATEADD(YEAR, 1, HireDate) as OneYearAfterHire,
    DATEADD(MONTH, -6, GETDATE()) as SixMonthsAgo,
    DATEADD(DAY, 30, HireDate) as ThirtyDaysAfterHire,
    
    -- Format dates
    FORMAT(HireDate, 'dd/MM/yyyy') as FormattedDate,
    FORMAT(HireDate, 'MMMM dd, yyyy') as LongDateFormat,
    CONVERT(VARCHAR, HireDate, 103) as DDMMYYYYFormat,
    CONVERT(VARCHAR, HireDate, 101) as MMDDYYYYFormat
FROM Employees;

-- Tìm nhân viên được tuyển trong 2 năm gần đây
SELECT * FROM Employees 
WHERE HireDate >= DATEADD(YEAR, -2, GETDATE());
```

## 15. NULL Handling

```sql
SELECT 
    FirstName,
    LastName,
    Phone,
    
    -- ISNULL - SQL Server specific
    ISNULL(Phone, 'No Phone') as PhoneWithDefault,
    
    -- COALESCE - ANSI standard (recommend)
    COALESCE(Phone, Email, 'No Contact') as ContactInfo,
    
    -- NULLIF
    NULLIF(Phone, '') as PhoneNullIfEmpty,
    
    -- IIF (SQL Server 2012+)
    IIF(Phone IS NULL, 'No Phone', 'Has Phone') as PhoneStatus,
    
    -- CASE for complex NULL handling
    CASE 
        WHEN Phone IS NULL THEN 'No Phone'
        WHEN LEN(Phone) < 10 THEN 'Invalid Phone'
        ELSE 'Valid Phone'
    END as PhoneValidation
FROM Employees;
```

## 16. Transactions (QUAN TRỌNG)

```sql
-- Transaction đơn giản
BEGIN TRANSACTION;

UPDATE Employees 
SET Salary = Salary * 1.1 
WHERE DepartmentID = 1;

-- Kiểm tra kết quả
IF @@ROWCOUNT > 0
    COMMIT TRANSACTION;
ELSE
    ROLLBACK TRANSACTION;

-- Transaction với Savepoint
BEGIN TRANSACTION MainTran;

INSERT INTO Departments (DepartmentName, Budget) 
VALUES ('R&D', 400000);

SAVE TRANSACTION SavePoint1;

INSERT INTO Employees (FirstName, LastName, Email, HireDate, Salary, DepartmentID)
VALUES ('New', 'Employee', 'new@company.com', GETDATE(), 15000000, @@IDENTITY);

-- Nếu có lỗi, rollback to savepoint
-- ROLLBACK TRANSACTION SavePoint1;

COMMIT TRANSACTION MainTran;
```

## 17. Indexes (Cải thiện Performance)

```sql
-- Tạo Index
CREATE INDEX IX_Employees_DepartmentID 
ON Employees (DepartmentID);

CREATE INDEX IX_Employees_Email 
ON Employees (Email);

-- Composite Index
CREATE INDEX IX_Employees_Name_Salary 
ON Employees (LastName, FirstName) 
INCLUDE (Salary);

-- Unique Index
CREATE UNIQUE INDEX IX_Employees_Email_Unique 
ON Employees (Email) 
WHERE Email IS NOT NULL;

-- Xem thông tin Index
SELECT 
    i.name as IndexName,
    i.type_desc as IndexType,
    c.name as ColumnName
FROM sys.indexes i
INNER JOIN sys.index_columns ic ON i.object_id = ic.object_id AND i.index_id = ic.index_id
INNER JOIN sys.columns c ON ic.object_id = c.object_id AND ic.column_id = c.column_id
WHERE i.object_id = OBJECT_ID('Employees')
ORDER BY i.name, ic.index_column_id;
```

## 18. Window Functions (NÂNG CAO)

```sql
SELECT 
    FirstName,
    LastName,
    Salary,
    DepartmentID,
    
    -- ROW_NUMBER
    ROW_NUMBER() OVER (ORDER BY Salary DESC) as SalaryRank,
    ROW_NUMBER() OVER (PARTITION BY DepartmentID ORDER BY Salary DESC) as DeptSalaryRank,
    
    -- RANK and DENSE_RANK
    RANK() OVER (ORDER BY Salary DESC) as SalaryRankWithTies,
    DENSE_RANK() OVER (ORDER BY Salary DESC) as SalaryDenseRank,
    
    -- NTILE
    NTILE(3) OVER (ORDER BY Salary DESC) as SalaryTertile,
    
    -- LAG and LEAD
    LAG(Salary, 1) OVER (ORDER BY EmployeeID) as PreviousSalary,
    LEAD(Salary, 1) OVER (ORDER BY EmployeeID) as NextSalary,
    
    -- Aggregate window functions
    SUM(Salary) OVER (PARTITION BY DepartmentID) as DeptTotalSalary,
    AVG(Salary) OVER (PARTITION BY DepartmentID) as DeptAvgSalary,
    COUNT(*) OVER (PARTITION BY DepartmentID) as DeptEmployeeCount
FROM Employees
ORDER BY DepartmentID, Salary DESC;
```

## 19. PIVOT và UNPIVOT

```sql
-- PIVOT - Chuyển từ rows thành columns
SELECT 
    DepartmentName,
    ISNULL([Junior], 0) as Junior,
    ISNULL([Middle], 0) as Middle,
    ISNULL([Senior], 0) as Senior
FROM (
    SELECT 
        d.DepartmentName,
        CASE 
            WHEN e.Salary >= 18000000 THEN 'Senior'
            WHEN e.Salary >= 15000000 THEN 'Middle'
            ELSE 'Junior'
        END as Level,
        e.EmployeeID
    FROM Employees e
    INNER JOIN Departments d ON e.DepartmentID = d.DepartmentID
) as SourceTable
PIVOT (
    COUNT(EmployeeID)
    FOR Level IN ([Junior], [Middle], [Senior])
) as PivotTable;
```

## 20. Dynamic SQL

```sql
-- Dynamic SQL cơ bản
DECLARE @SQL NVARCHAR(MAX);
DECLARE @TableName NVARCHAR(100) = 'Employees';
DECLARE @MinSalary DECIMAL(10,2) = 15000000;

SET @SQL = N'SELECT * FROM ' + QUOTENAME(@TableName) + 
           N' WHERE Salary >= @MinSalaryParam';

EXEC sp_executesql @SQL, 
                   N'@MinSalaryParam DECIMAL(10,2)', 
                   @MinSalaryParam = @MinSalary;
```

## 21. Backup và Restore

```sql
-- Full Backup
BACKUP DATABASE CompanyDB 
TO DISK = 'C:\Backup\CompanyDB_Full.bak'
WITH FORMAT, INIT, NAME = 'Full Backup of CompanyDB';

-- Differential Backup
BACKUP DATABASE CompanyDB 
TO DISK = 'C:\Backup\CompanyDB_Diff.bak'
WITH DIFFERENTIAL, FORMAT, INIT, NAME = 'Differential Backup of CompanyDB';

-- Transaction Log Backup
BACKUP LOG CompanyDB 
TO DISK = 'C:\Backup\CompanyDB_Log.trn'
WITH FORMAT, INIT, NAME = 'Log Backup of CompanyDB';

-- Restore Database (ví dụ)
/*
RESTORE DATABASE CompanyDB_Test 
FROM DISK = 'C:\Backup\CompanyDB_Full.bak'
WITH MOVE 'CompanyDB' TO 'C:\Data\CompanyDB_Test.mdf',
     MOVE 'CompanyDB_Log' TO 'C:\Data\CompanyDB_Test.ldf',
     REPLACE;
*/
```

## 22. Permissions và Security

```sql
-- Tạo Login (commented - cần admin rights)
-- CREATE LOGIN AppUser WITH PASSWORD = 'StrongPassword123!';

-- Tạo User trong database
-- CREATE USER AppUser FOR LOGIN AppUser;

-- Grant permissions
-- GRANT SELECT, INSERT, UPDATE ON Employees TO AppUser;
-- GRANT EXECUTE ON GetEmployeesByDepartment TO AppUser;

-- Create Role
-- CREATE ROLE EmployeeReader;
-- GRANT SELECT ON Employees TO EmployeeReader;
-- ALTER ROLE EmployeeReader ADD MEMBER AppUser;
```

## 23. Performance Monitoring

```sql
-- Xem thống kê IO
SET STATISTICS IO ON;
SELECT e.*, d.DepartmentName 
FROM Employees e
INNER JOIN Departments d ON e.DepartmentID = d.DepartmentID;
SET STATISTICS IO OFF;

-- Query để tìm expensive queries
SELECT TOP 10
    qs.total_elapsed_time / qs.execution_count as avg_elapsed_time,
    qs.total_worker_time / qs.execution_count as avg_cpu_time,
    qs.execution_count,
    SUBSTRING(qt.text, (qs.statement_start_offset/2)+1, 
        ((CASE qs.statement_end_offset
          WHEN -1 THEN DATALENGTH(qt.text)
          ELSE qs.statement_end_offset
          END - qs.statement_start_offset)/2)+1) as query_text
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) qt
ORDER BY avg_elapsed_time DESC;
```

## 24. System Queries hữu ích

```sql
-- Xem thông tin database
SELECT 
    name as DatabaseName,
    database_id,
    create_date,
    collation_name,
    state_desc as Status
FROM sys.databases;

-- Xem size của tables
SELECT 
    t.NAME AS TableName,
    s.Name AS SchemaName,
    p.rows AS RowCounts,
    CAST(ROUND(((SUM(a.total_pages) * 8) / 1024.00), 2) AS NUMERIC(36, 2)) AS TotalSpaceMB,
    CAST(ROUND(((SUM(a.used_pages) * 8) / 1024.00), 2) AS NUMERIC(36, 2)) AS UsedSpaceMB,
    CAST(ROUND(((SUM(a.total_pages) - SUM(a.used_pages)) * 8) / 1024.00, 2) AS NUMERIC(36, 2)) AS UnusedSpaceMB
FROM sys.tables t
INNER JOIN sys.indexes i ON t.OBJECT_ID = i.object_id
INNER JOIN sys.partitions p ON i.object_id = p.OBJECT_ID AND i.index_id = p.index_id
INNER JOIN sys.allocation_units a ON p.partition_id = a.container_id
LEFT OUTER JOIN sys.schemas s ON t.schema_id = s.schema_id
WHERE t.NAME NOT LIKE 'dt%' 
    AND t.is_ms_shipped = 0
    AND i.OBJECT_ID > 255
GROUP BY t.Name, s.Name, p.Rows
ORDER BY TotalSpaceMB DESC;

-- Xem thông tin về foreign keys
SELECT 
    fk.name AS ForeignKeyName,
    OBJECT_SCHEMA_NAME(fk.parent_object_id) AS SchemaName,
    OBJECT_NAME(fk.parent_object_id) AS TableName,
    c1.name AS ColumnName,
    OBJECT_SCHEMA_NAME(fk.referenced_object_id) AS ReferencedSchemaName,
    OBJECT_NAME(fk.referenced_object_id) AS ReferencedTableName,
    c2.name AS ReferencedColumnName
FROM sys.foreign_keys fk
INNER JOIN sys.foreign_key_columns fkc ON fk.object_id = fkc.constraint_object_id
INNER JOIN sys.columns c1 ON fkc.parent_object_id = c1.object_id AND fkc.parent_column_id = c1.column_id
INNER JOIN sys.columns c2 ON fkc.referenced_object_id = c2.object_id AND fkc.referenced_column_id = c2.column_id
ORDER BY SchemaName, TableName;
```

## 25. Common Table Expressions (CTE) - Nâng cao

```sql
-- Recursive CTE - Tạo số từ 1 đến 10
WITH Numbers AS (
    SELECT 1 as n
    UNION ALL
    SELECT n + 1
    FROM Numbers
    WHERE n < 10
)
SELECT n FROM Numbers;

-- Recursive CTE - Organizational Hierarchy (nếu có manager_id)
/*
WITH EmployeeHierarchy AS (
    -- Anchor: Top level managers
    SELECT 
        EmployeeID,
        FirstName + ' ' + LastName as FullName,
        ManagerID,
        0 as Level,
        CAST(FirstName + ' ' + LastName AS NVARCHAR(1000)) as Hierarchy
    FROM Employees 
    WHERE ManagerID IS NULL
    
    UNION ALL
    
    -- Recursive: Subordinates
    SELECT 
        e.EmployeeID,
        e.FirstName + ' ' + e.LastName,
        e.ManagerID,
        eh.Level + 1,
        CAST(eh.Hierarchy + ' -> ' + e.FirstName + ' ' + e.LastName AS NVARCHAR(1000))
    FROM Employees e
    INNER JOIN EmployeeHierarchy eh ON e.ManagerID = eh.EmployeeID
)
SELECT * FROM EmployeeHierarchy ORDER BY Level, FullName;
*/
```

## 26. XML Functions

```sql
-- Tạo XML từ data
SELECT 
    EmployeeID,
    FirstName,
    LastName,
    Salary
FROM Employees
FOR XML PATH('Employee'), ROOT('Employees');

-- Parse XML (giả sử có XML data)
DECLARE @xmldata XML = '
<Employees>
    <Employee>
        <FirstName>John</FirstName>
        <LastName>Doe</LastName>
        <Salary>50000</Salary>
    </Employee>
</Employees>';

SELECT 
    T.c.value('FirstName[1]', 'NVARCHAR(50)') as FirstName,
    T.c.value('LastName[1]', 'NVARCHAR(50)') as LastName,
    T.c.value('Salary[1]', 'DECIMAL(10,2)') as Salary
FROM @xmldata.nodes('/Employees/Employee') T(c);
```

## 27. JSON Functions

```sql
-- Convert to JSON (SQL Server 2016+)
SELECT 
    EmployeeID,
    FirstName,
    LastName,
    Salary,
    HireDate
FROM Employees
FOR JSON PATH;

-- Parse JSON
DECLARE @json NVARCHAR(MAX) = N'[
    {"FirstName":"John","LastName":"Doe","Salary":50000},
    {"FirstName":"Jane","LastName":"Smith","Salary":60000}
]';

SELECT * FROM OPENJSON(@json)
WITH (
    FirstName NVARCHAR(50),
    LastName NVARCHAR(50),
    Salary DECIMAL(10,2)
);

-- JSON functions
SELECT 
    EmployeeID,
    FirstName,
    LastName,
    JSON_VALUE('{"department":"IT","level":"Senior"}', '$.department') as Department,
    JSON_QUERY('{"skills":["C#","SQL","JavaScript"]}', '$.skills') as Skills
FROM Employees
WHERE EmployeeID = 1;
```

## 28. Temporary Tables và Table Variables

```sql
-- Table Variable (scope limited, small data)
DECLARE @TempEmployees TABLE (
    EmployeeID INT,
    FullName NVARCHAR(101),
    Salary DECIMAL(10,2)
);

INSERT INTO @TempEmployees
SELECT EmployeeID, FirstName + ' ' + LastName, Salary
FROM Employees
WHERE Salary > 15000000;

SELECT * FROM @TempEmployees;

-- Local Temporary Table (session scope)
CREATE TABLE #TempEmployeeStats (
    DepartmentID INT,
    DepartmentName NVARCHAR(100),
    EmployeeCount INT,
    AvgSalary DECIMAL(10,2),
    MaxSalary DECIMAL(10,2)
);

INSERT INTO #TempEmployeeStats
SELECT 
    d.DepartmentID,
    d.DepartmentName,
    COUNT(e.EmployeeID),
    AVG(e.Salary),
    MAX(e.Salary)
FROM Departments d
LEFT JOIN Employees e ON d.DepartmentID = e.DepartmentID
GROUP BY d.DepartmentID, d.DepartmentName;

SELECT * FROM #TempEmployeeStats;
```

## 29. Cursors

> **⚠️ Lưu ý**: Tốt nhất tránh dùng Cursors, sử dụng set-based operations thay thế

```sql
-- Cursor example (avoid if possible)
DECLARE @EmployeeID INT, @FullName NVARCHAR(101), @Salary DECIMAL(10,2);

DECLARE employee_cursor CURSOR FOR
SELECT EmployeeID, FirstName + ' ' + LastName, Salary
FROM Employees
WHERE IsActive = 1;

OPEN employee_cursor;

FETCH NEXT FROM employee_cursor INTO @EmployeeID, @FullName, @Salary;

WHILE @@FETCH_STATUS = 0
BEGIN
    PRINT 'Employee: ' + @FullName + ', Salary: ' + CAST(@Salary AS NVARCHAR(20));
    
    -- Process individual row here
    
    FETCH NEXT FROM employee_cursor INTO @EmployeeID, @FullName, @Salary;
END

CLOSE employee_cursor;
DEALLOCATE employee_cursor;
```

## 30. MERGE Statement (UPSERT)

```sql
-- Create a staging table for demo
CREATE TABLE EmployeeUpdates (
    EmployeeID INT,
    FirstName NVARCHAR(50),
    LastName NVARCHAR(50),
    Salary DECIMAL(10,2)
);

-- Insert some test data
INSERT INTO EmployeeUpdates VALUES 
(1, 'Nguyen', 'Van A', 16000000),  -- Update existing
(99, 'New', 'Employee', 14000000); -- Insert new

-- MERGE statement
MERGE Employees AS target
USING EmployeeUpdates AS source
ON target.EmployeeID = source.EmployeeID

WHEN MATCHED THEN
    UPDATE SET 
        FirstName = source.FirstName,
        LastName = source.LastName,
        Salary = source.Salary

WHEN NOT MATCHED BY TARGET THEN
    INSERT (FirstName, LastName, Salary, HireDate, DepartmentID, IsActive)
    VALUES (source.FirstName, source.LastName, source.Salary, GETDATE(), 1, 1)

WHEN NOT MATCHED BY SOURCE THEN
    UPDATE SET IsActive = 0

OUTPUT $action as Action, 
       INSERTED.EmployeeID, 
       INSERTED.FirstName + ' ' + INSERTED.LastName as FullName;

-- Cleanup
DROP TABLE EmployeeUpdates;
```

## 31. Common Patterns và Best Practices

### Pagination Pattern

```sql
-- Method 1: ROW_NUMBER()
DECLARE @PageNumber INT = 1;
DECLARE @PageSize INT = 10;

SELECT *
FROM (
    SELECT 
        *,
        ROW_NUMBER() OVER (ORDER BY EmployeeID) as RowNum
    FROM Employees
    WHERE IsActive = 1
) as PagedData
WHERE RowNum BETWEEN (@PageNumber - 1) * @PageSize + 1 
                 AND @PageNumber * @PageSize;

-- Method 2: OFFSET/FETCH (SQL Server 2012+) - Preferred
SELECT *
FROM Employees
WHERE IsActive = 1
ORDER BY EmployeeID
OFFSET ((@PageNumber - 1) * @PageSize) ROWS
FETCH NEXT @PageSize ROWS ONLY;
```

### Find và Remove Duplicates

```sql
-- Find duplicates
SELECT 
    FirstName,
    LastName,
    COUNT(*) as DuplicateCount
FROM Employees
GROUP BY FirstName, LastName
HAVING COUNT(*) > 1;

-- Remove duplicates (keep latest)
WITH DuplicateCTE AS (
    SELECT *,
        ROW_NUMBER() OVER (
            PARTITION BY FirstName, LastName 
            ORDER BY EmployeeID DESC
        ) as rn
    FROM Employees
)
DELETE FROM DuplicateCTE WHERE rn > 1;
```

### Running Totals

```sql
SELECT 
    EmployeeID,
    FirstName,
    LastName,
    Salary,
    SUM(Salary) OVER (ORDER BY EmployeeID ROWS UNBOUNDED PRECEDING) as RunningTotal
FROM Employees
ORDER BY EmployeeID;
```

### Conditional Aggregation

```sql
SELECT 
    DepartmentID,
    COUNT(*) as TotalEmployees,
    COUNT(CASE WHEN Salary >= 15000000 THEN 1 END) as HighSalaryEmployees,
    COUNT(CASE WHEN DATEDIFF(YEAR, HireDate, GETDATE()) >= 3 THEN 1 END) as ExperiencedEmployees,
    AVG(CASE WHEN Salary >= 15000000 THEN Salary END) as AvgHighSalary
FROM Employees
GROUP BY DepartmentID;
```

## 32. Performance Tips & Tricks

```sql
-- 1. Use EXISTS instead of IN for better performance
SELECT * FROM Employees e
WHERE EXISTS (
    SELECT 1 FROM EmployeeProjects ep 
    WHERE ep.EmployeeID = e.EmployeeID
);

-- 2. Use UNION ALL instead of UNION when duplicates are acceptable
SELECT FirstName, LastName FROM Employees WHERE DepartmentID = 1
UNION ALL
SELECT FirstName, LastName FROM Employees WHERE DepartmentID = 2;

-- 3. Use WITH (NOLOCK) carefully (dirty reads)
SELECT * FROM Employees WITH (NOLOCK)
WHERE Salary > 15000000;

-- 4. Use specific columns instead of *
SELECT EmployeeID, FirstName, LastName, Salary 
FROM Employees  -- Better than SELECT *

-- 5. Use indexed columns in WHERE clauses
SELECT * FROM Employees 
WHERE DepartmentID = 1  -- Assuming index on DepartmentID
ORDER BY EmployeeID;    -- Assuming clustered index on EmployeeID
```

## 33. Error Handling Patterns

```sql
-- Pattern: Comprehensive validation
CREATE PROCEDURE CreateEmployee
    @FirstName NVARCHAR(50),
    @LastName NVARCHAR(50),
    @Email NVARCHAR(100),
    @Salary DECIMAL(10,2),
    @DepartmentID INT
AS
BEGIN
    SET NOCOUNT ON;
    
    -- Validation
    IF @FirstName IS NULL OR LTRIM(RTRIM(@FirstName)) = ''
    BEGIN
        RAISERROR('FirstName is required', 16, 1);
        RETURN;
    END
    
    IF @Email IS NULL OR @Email NOT LIKE '%@%.%'
    BEGIN
        RAISERROR('Valid email is required', 16, 1);
        RETURN;
    END
    
    IF NOT EXISTS (SELECT 1 FROM Departments WHERE DepartmentID = @DepartmentID)
    BEGIN
        RAISERROR('Invalid Department ID', 16, 1);
        RETURN;
    END
    
    -- Insert
    BEGIN TRY
        INSERT INTO Employees (FirstName, LastName, Email, Salary, DepartmentID, HireDate, IsActive)
        VALUES (@FirstName, @LastName, @Email, @Salary, @DepartmentID, GETDATE(), 1);
        
        SELECT SCOPE_IDENTITY() as NewEmployeeID;
        
    END TRY
    BEGIN CATCH
        IF ERROR_NUMBER() = 2627 -- Duplicate key
            RAISERROR('Email already exists', 16, 1);
        ELSE
            THROW;
    END CATCH
END
GO
```

## 34. Maintenance Commands

```sql
-- Update statistics
UPDATE STATISTICS Employees;

-- Rebuild all indexes on a table
ALTER INDEX ALL ON Employees REBUILD;

-- Reorganize fragmented indexes
ALTER INDEX ALL ON Employees REORGANIZE;

-- Check database integrity
DBCC CHECKDB('CompanyDB');

-- View fragmentation
SELECT 
    OBJECT_NAME(ips.object_id) AS TableName,
    i.name AS IndexName,
    ips.index_type_desc,
    ips.avg_fragmentation_in_percent,
    ips.page_count
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'DETAILED') ips
INNER JOIN sys.indexes i ON ips.object_id = i.object_id AND ips.index_id = i.index_id
WHERE ips.avg_fragmentation_in_percent > 10
ORDER BY ips.avg_fragmentation_in_percent DESC;
```

## 35. Common Business Scenarios

### Department Budget Utilization

```sql
SELECT 
    d.DepartmentName,
    d.Budget as AllocatedBudget,
    ISNULL(SUM(e.Salary), 0) as UsedBudget,
    d.Budget - ISNULL(SUM(e.Salary), 0) as RemainingBudget,
    CASE 
        WHEN d.Budget > 0 THEN 
            CAST((ISNULL(SUM(e.Salary), 0) / d.Budget) * 100 AS DECIMAL(5,2))
        ELSE 0 
    END as UtilizationPercent
FROM Departments d
LEFT JOIN Employees e ON d.DepartmentID = e.DepartmentID AND e.IsActive = 1
GROUP BY d.DepartmentID, d.DepartmentName, d.Budget;
```

### Employee Anniversary Report

```sql
SELECT 
    FirstName + ' ' + LastName as EmployeeName,
    HireDate,
    DATEDIFF(YEAR, HireDate, GETDATE()) as YearsOfService,
    CASE 
        WHEN DATEDIFF(YEAR, HireDate, GETDATE()) = 1 THEN '1 Year'
        WHEN DATEDIFF(YEAR, HireDate, GETDATE()) = 5 THEN '5 Years'
        WHEN DATEDIFF(YEAR, HireDate, GETDATE()) = 10 THEN '10 Years'
        WHEN DATEDIFF(YEAR, HireDate, GETDATE()) % 5 = 0 THEN 
            CAST(DATEDIFF(YEAR, HireDate, GETDATE()) AS VARCHAR) + ' Years'
    END as Milestone
FROM Employees
WHERE IsActive = 1
    AND (
        DATEDIFF(YEAR, HireDate, GETDATE()) = 1 OR
        DATEDIFF(YEAR, HireDate, GETDATE()) % 5 = 0
    )
ORDER BY YearsOfService DESC;
```

---

## 🎯 Tổng kết những điều QUAN TRỌNG NHẤT

### 🔥 Mức độ BẮT BUỘC (dùng hàng ngày)
1. **SELECT, INSERT, UPDATE, DELETE** với các điều kiện
2. **JOINs** (INNER, LEFT, RIGHT, FULL)
3. **GROUP BY, HAVING, ORDER BY**
4. **Subqueries và CTEs**
5. **CASE WHEN** cho logic điều kiện
6. **String và Date functions**
7. **NULL handling** (ISNULL, COALESCE)

### 🚀 Mức độ THƯỜNG XUYÊN
8. **Stored Procedures và Functions**
9. **Views**
10. **Error handling** với TRY/CATCH
11. **Transactions**
12. **Variables và control flow** (IF/ELSE, WHILE)
13. **Indexes** cơ bản

### ⚡ Mức độ HỮU ÍCH khi cần
14. **Window functions** cơ bản
15. **Performance monitoring** và optimization
16. **Backup/Restore** procedures
17. **Basic security** và permissions

### 🛠️ Những công cụ PHẢI BIẾT
- **SQL Server Management Studio (SSMS)**
- **Execution Plans** để optimize queries
- **Azure Data Studio** (ngày càng phổ biến)
- **Visual Studio Code** với SQL extensions

### 📝 Lưu ý khi ĐI LÀM
- ✅ Luôn test trên database development trước
- ✅ Backup trước khi chạy scripts quan trọng  
- ✅ Comment code rõ ràng
- ✅ Follow naming conventions của công ty
- ✅ Optimize queries cho performance
- ✅ Handle errors properly
- ✅ Sử dụng transactions khi cần thiết
- ✅ Tránh SELECT * trong production code

> **💡 Pro Tip**: Bookmark tài liệu này và thực hành từng phần theo thứ tự ưu tiên. Với bộ kiến thức này, bạn sẽ handle được 95% công việc SQL Server trong thực tế!