# Microsoft SQL Server: Stored Procedures, Triggers & Functions

A comprehensive guide to understanding and implementing stored procedures, triggers, and functions in Microsoft SQL Server.

## Table of Contents
- [Overview](#overview)
- [Stored Procedures](#stored-procedures)
- [Functions](#functions)
- [Triggers](#triggers)
- [Best Practices](#best-practices)
- [Common Use Cases](#common-use-cases)

## Overview

SQL Server provides three main types of programmable objects that help encapsulate business logic and improve database performance:

- **Stored Procedures**: Precompiled SQL statements that can accept parameters and perform complex operations
- **Functions**: Return a single value or table and can be used in SQL expressions
- **Triggers**: Special stored procedures that automatically execute in response to database events

## Stored Procedures

Stored procedures are precompiled collections of SQL statements that can accept input parameters, perform operations, and return results.

### Basic Syntax
```sql
CREATE PROCEDURE procedure_name
    @parameter1 datatype,
    @parameter2 datatype = default_value
AS
BEGIN
    -- SQL statements
END
```

### Example 1: Simple Stored Procedure
```sql
-- Create a procedure to get employee details
CREATE PROCEDURE GetEmployeeById
    @EmployeeId INT
AS
BEGIN
    SELECT 
        EmployeeId,
        FirstName,
        LastName,
        Email,
        Salary
    FROM Employees
    WHERE EmployeeId = @EmployeeId
END

-- Execute the procedure
EXEC GetEmployeeById @EmployeeId = 123
```

### Example 2: Stored Procedure with Multiple Parameters and Output
```sql
CREATE PROCEDURE UpdateEmployeeSalary
    @EmployeeId INT,
    @NewSalary DECIMAL(10,2),
    @UpdatedRows INT OUTPUT
AS
BEGIN
    UPDATE Employees
    SET Salary = @NewSalary,
        ModifiedDate = GETDATE()
    WHERE EmployeeId = @EmployeeId
    
    SET @UpdatedRows = @@ROWCOUNT
    
    IF @UpdatedRows > 0
        PRINT 'Salary updated successfully'
    ELSE
        PRINT 'Employee not found'
END

-- Execute with output parameter
DECLARE @RowsAffected INT
EXEC UpdateEmployeeSalary 
    @EmployeeId = 123,
    @NewSalary = 75000.00,
    @UpdatedRows = @RowsAffected OUTPUT

PRINT 'Rows affected: ' + CAST(@RowsAffected AS VARCHAR(10))
```

### Example 3: Complex Stored Procedure with Error Handling
```sql
CREATE PROCEDURE CreateNewEmployee
    @FirstName NVARCHAR(50),
    @LastName NVARCHAR(50),
    @Email NVARCHAR(100),
    @DepartmentId INT,
    @Salary DECIMAL(10,2),
    @NewEmployeeId INT OUTPUT
AS
BEGIN
    SET NOCOUNT ON
    
    BEGIN TRY
        BEGIN TRANSACTION
        
        -- Check if email already exists
        IF EXISTS (SELECT 1 FROM Employees WHERE Email = @Email)
        BEGIN
            RAISERROR('Email already exists', 16, 1)
            RETURN
        END
        
        -- Insert new employee
        INSERT INTO Employees (FirstName, LastName, Email, DepartmentId, Salary, CreatedDate)
        VALUES (@FirstName, @LastName, @Email, @DepartmentId, @Salary, GETDATE())
        
        SET @NewEmployeeId = SCOPE_IDENTITY()
        
        -- Log the action
        INSERT INTO AuditLog (Action, TableName, RecordId, CreatedDate)
        VALUES ('INSERT', 'Employees', @NewEmployeeId, GETDATE())
        
        COMMIT TRANSACTION
        PRINT 'Employee created successfully with ID: ' + CAST(@NewEmployeeId AS VARCHAR(10))
        
    END TRY
    BEGIN CATCH
        ROLLBACK TRANSACTION
        
        DECLARE @ErrorMessage NVARCHAR(4000) = ERROR_MESSAGE()
        DECLARE @ErrorSeverity INT = ERROR_SEVERITY()
        DECLARE @ErrorState INT = ERROR_STATE()
        
        RAISERROR(@ErrorMessage, @ErrorSeverity, @ErrorState)
    END CATCH
END
```

## Functions

Functions return a value and can be used in SQL expressions. SQL Server supports scalar functions (return single value) and table-valued functions (return table).

### Scalar Functions

#### Example 1: Simple Scalar Function
```sql
CREATE FUNCTION CalculateAge(@BirthDate DATE)
RETURNS INT
AS
BEGIN
    DECLARE @Age INT
    SET @Age = DATEDIFF(YEAR, @BirthDate, GETDATE())
    
    -- Adjust for birthday not yet occurred this year
    IF DATEADD(YEAR, @Age, @BirthDate) > GETDATE()
        SET @Age = @Age - 1
    
    RETURN @Age
END

-- Usage
SELECT 
    FirstName,
    LastName,
    BirthDate,
    dbo.CalculateAge(BirthDate) AS Age
FROM Employees
```

#### Example 2: Scalar Function with Complex Logic
```sql
CREATE FUNCTION GetEmployeeGrade(@Salary DECIMAL(10,2), @YearsOfService INT)
RETURNS CHAR(1)
AS
BEGIN
    DECLARE @Grade CHAR(1)
    
    IF @Salary >= 100000 AND @YearsOfService >= 10
        SET @Grade = 'A'
    ELSE IF @Salary >= 75000 AND @YearsOfService >= 5
        SET @Grade = 'B'
    ELSE IF @Salary >= 50000 AND @YearsOfService >= 2
        SET @Grade = 'C'
    ELSE
        SET @Grade = 'D'
    
    RETURN @Grade
END

-- Usage
SELECT 
    FirstName + ' ' + LastName AS FullName,
    Salary,
    YearsOfService,
    dbo.GetEmployeeGrade(Salary, YearsOfService) AS Grade
FROM Employees
```

### Table-Valued Functions

#### Example 1: Inline Table-Valued Function
```sql
CREATE FUNCTION GetEmployeesByDepartment(@DepartmentId INT)
RETURNS TABLE
AS
RETURN
(
    SELECT 
        e.EmployeeId,
        e.FirstName,
        e.LastName,
        e.Email,
        e.Salary,
        d.DepartmentName
    FROM Employees e
    INNER JOIN Departments d ON e.DepartmentId = d.DepartmentId
    WHERE e.DepartmentId = @DepartmentId
)

-- Usage
SELECT * FROM dbo.GetEmployeesByDepartment(1)
```

#### Example 2: Multi-Statement Table-Valued Function
```sql
CREATE FUNCTION GetEmployeeSalaryHistory(@EmployeeId INT)
RETURNS @SalaryHistory TABLE
(
    EmployeeId INT,
    Salary DECIMAL(10,2),
    EffectiveDate DATE,
    SalaryChange DECIMAL(10,2),
    ChangePercentage DECIMAL(5,2)
)
AS
BEGIN
    INSERT INTO @SalaryHistory (EmployeeId, Salary, EffectiveDate, SalaryChange, ChangePercentage)
    SELECT 
        EmployeeId,
        Salary,
        EffectiveDate,
        Salary - LAG(Salary) OVER (ORDER BY EffectiveDate) AS SalaryChange,
        CASE 
            WHEN LAG(Salary) OVER (ORDER BY EffectiveDate) IS NOT NULL 
            THEN ((Salary - LAG(Salary) OVER (ORDER BY EffectiveDate)) / LAG(Salary) OVER (ORDER BY EffectiveDate)) * 100
            ELSE 0
        END AS ChangePercentage
    FROM SalaryHistory
    WHERE EmployeeId = @EmployeeId
    
    RETURN
END

-- Usage
SELECT * FROM dbo.GetEmployeeSalaryHistory(123)
```

## Triggers

Triggers are special stored procedures that automatically execute in response to database events.

### DML Triggers (Data Manipulation Language)

#### Example 1: AFTER INSERT Trigger
```sql
CREATE TRIGGER trg_Employee_AfterInsert
ON Employees
AFTER INSERT
AS
BEGIN
    SET NOCOUNT ON
    
    -- Log new employee creation
    INSERT INTO AuditLog (Action, TableName, RecordId, NewValues, CreatedDate, CreatedBy)
    SELECT 
        'INSERT',
        'Employees',
        i.EmployeeId,
        'FirstName: ' + i.FirstName + ', LastName: ' + i.LastName + ', Email: ' + i.Email,
        GETDATE(),
        SYSTEM_USER
    FROM inserted i
    
    -- Send welcome email notification (placeholder)
    INSERT INTO EmailQueue (ToEmail, Subject, Body, CreatedDate)
    SELECT 
        i.Email,
        'Welcome to the Company!',
        'Dear ' + i.FirstName + ', Welcome to our organization!',
        GETDATE()
    FROM inserted i
END
```

#### Example 2: AFTER UPDATE Trigger
```sql
CREATE TRIGGER trg_Employee_AfterUpdate
ON Employees
AFTER UPDATE
AS
BEGIN
    SET NOCOUNT ON
    
    -- Only log if specific columns were updated
    IF UPDATE(Salary) OR UPDATE(DepartmentId)
    BEGIN
        INSERT INTO AuditLog (Action, TableName, RecordId, OldValues, NewValues, CreatedDate, CreatedBy)
        SELECT 
            'UPDATE',
            'Employees',
            i.EmployeeId,
            'Salary: ' + CAST(d.Salary AS VARCHAR(20)) + ', DepartmentId: ' + CAST(d.DepartmentId AS VARCHAR(10)),
            'Salary: ' + CAST(i.Salary AS VARCHAR(20)) + ', DepartmentId: ' + CAST(i.DepartmentId AS VARCHAR(10)),
            GETDATE(),
            SYSTEM_USER
        FROM inserted i
        INNER JOIN deleted d ON i.EmployeeId = d.EmployeeId
        
        -- If salary changed, insert into salary history
        IF UPDATE(Salary)
        BEGIN
            INSERT INTO SalaryHistory (EmployeeId, OldSalary, NewSalary, EffectiveDate, ChangedBy)
            SELECT 
                i.EmployeeId,
                d.Salary,
                i.Salary,
                GETDATE(),
                SYSTEM_USER
            FROM inserted i
            INNER JOIN deleted d ON i.EmployeeId = d.EmployeeId
            WHERE i.Salary != d.Salary
        END
    END
END
```

#### Example 3: INSTEAD OF Trigger (for Views)
```sql
-- Create a view first
CREATE VIEW vw_EmployeeDetails AS
SELECT 
    e.EmployeeId,
    e.FirstName,
    e.LastName,
    e.Email,
    e.Salary,
    d.DepartmentName,
    d.DepartmentId
FROM Employees e
INNER JOIN Departments d ON e.DepartmentId = d.DepartmentId

-- Create INSTEAD OF trigger
CREATE TRIGGER trg_EmployeeDetails_InsteadOfUpdate
ON vw_EmployeeDetails
INSTEAD OF UPDATE
AS
BEGIN
    SET NOCOUNT ON
    
    -- Update Employees table
    UPDATE e
    SET 
        FirstName = i.FirstName,
        LastName = i.LastName,
        Email = i.Email,
        Salary = i.Salary,
        DepartmentId = i.DepartmentId,
        ModifiedDate = GETDATE()
    FROM Employees e
    INNER JOIN inserted i ON e.EmployeeId = i.EmployeeId
    
    PRINT 'Employee details updated through view'
END
```

### DDL Triggers (Data Definition Language)

#### Example: Database-level DDL Trigger
```sql
CREATE TRIGGER trg_DatabaseDDL
ON DATABASE
FOR CREATE_TABLE, ALTER_TABLE, DROP_TABLE
AS
BEGIN
    SET NOCOUNT ON
    
    DECLARE @EventData XML = EVENTDATA()
    
    INSERT INTO DDLAuditLog (
        EventType,
        DatabaseName,
        SchemaName,
        ObjectName,
        LoginName,
        EventDate,
        EventData
    )
    VALUES (
        @EventData.value('(/EVENT_INSTANCE/EventType)[1]', 'NVARCHAR(100)'),
        @EventData.value('(/EVENT_INSTANCE/DatabaseName)[1]', 'NVARCHAR(100)'),
        @EventData.value('(/EVENT_INSTANCE/SchemaName)[1]', 'NVARCHAR(100)'),
        @EventData.value('(/EVENT_INSTANCE/ObjectName)[1]', 'NVARCHAR(100)'),
        @EventData.value('(/EVENT_INSTANCE/LoginName)[1]', 'NVARCHAR(100)'),
        GETDATE(),
        @EventData
    )
END
```

## Best Practices

### Stored Procedures
- Use meaningful names with consistent naming conventions
- Always use SET NOCOUNT ON to improve performance
- Implement proper error handling with TRY-CATCH blocks
- Use transactions for multiple related operations
- Validate input parameters
- Use OUTPUT parameters instead of SELECT for returning single values

### Functions
- Keep functions simple and focused on a single task
- Avoid functions that modify data (use stored procedures instead)
- Be cautious with table-valued functions in WHERE clauses (performance impact)
- Consider inline table-valued functions over multi-statement when possible
- Functions should be deterministic when possible

### Triggers
- Keep trigger logic simple and fast
- Avoid recursive triggers unless necessary
- Handle multiple row operations (consider inserted/deleted tables)
- Be careful with triggers on frequently updated tables
- Document trigger dependencies clearly
- Test triggers thoroughly with batch operations

### General Guidelines
- Use appropriate data types and sizes
- Comment complex logic
- Follow consistent formatting and naming conventions
- Test with various scenarios including edge cases
- Monitor performance and optimize as needed
- Use version control for database objects

## Common Use Cases

### Stored Procedures
- Complex business logic implementation
- Data validation and processing
- Batch operations
- Report generation
- ETL processes

### Functions
- Data transformation and formatting
- Complex calculations
- Reusable logic across queries
- Data validation rules
- Custom aggregations

### Triggers
- Audit logging
- Data synchronization
- Business rule enforcement
- Automatic calculations
- Data archiving
- Security monitoring

## Performance Considerations

- **Stored Procedures**: Generally offer better performance due to query plan caching
- **Functions**: Scalar functions can impact performance when used in WHERE clauses
- **Triggers**: Should be lightweight; heavy operations should be moved to stored procedures called asynchronously
- Use appropriate indexing strategies
- Monitor execution plans and optimize accordingly
- Consider using table variables vs temp tables based on data size

## Conclusion

Stored procedures, functions, and triggers are powerful tools in SQL Server that help implement business logic, maintain data integrity, and improve application performance. Choose the right tool based on your specific requirements and follow best practices for optimal results.
