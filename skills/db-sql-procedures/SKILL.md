---
name: db-sql-procedures
description: Understand SQL Server stored procedures, T-SQL patterns, and SQL syntax best practices.
---

# SQL & Stored Procedures Skill

Apply this knowledge when analyzing SQL Server stored procedures, understanding T-SQL logic, and writing efficient database code.

## T-SQL Fundamentals

### Stored Procedure Structure

```sql
CREATE PROCEDURE [Schema].[ProcedureName]
    @InputParam1 INT,
    @InputParam2 VARCHAR(100),
    @OutputParam INT OUTPUT
AS
BEGIN
    SET NOCOUNT ON;

    BEGIN TRY
        BEGIN TRANSACTION;

        -- Business logic here

        COMMIT TRANSACTION;
    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0
            ROLLBACK TRANSACTION;

        -- Error handling
        THROW;
    END CATCH
END
```

### Common T-SQL Patterns

#### Conditional Logic
```sql
-- IF/ELSE
IF @Status = 1
BEGIN
    -- Multiple statements
END
ELSE IF @Status = 2
BEGIN
    -- Alternative
END

-- CASE expressions (inline)
SELECT
    CASE
        WHEN Score > 1000 THEN 'Expert'
        WHEN Score > 500 THEN 'Intermediate'
        ELSE 'Beginner'
    END AS PlayerRank
FROM Players

-- CASE for assignments
SET @GameMode = CASE @Type
    WHEN 'S' THEN 'Single'
    WHEN 'M' THEN 'Multiplayer'
    WHEN 'C' THEN 'Coop'
END
```

#### Cursor Patterns (Row-by-Row Processing)
```sql
DECLARE @SessionId INT
DECLARE session_cursor CURSOR FOR
    SELECT SessionId FROM GameSessions WHERE Status = 1

OPEN session_cursor
FETCH NEXT FROM session_cursor INTO @SessionId

WHILE @@FETCH_STATUS = 0
BEGIN
    -- Process each row
    EXEC ProcessSession @SessionId

    FETCH NEXT FROM session_cursor INTO @SessionId
END

CLOSE session_cursor
DEALLOCATE session_cursor
```

#### Temporary Tables
```sql
-- Local temp table (session scope)
CREATE TABLE #TempScores (
    PlayerId INT,
    TotalScore DECIMAL(18,2)
)

-- Table variable (batch scope)
DECLARE @Results TABLE (
    PlayerId INT,
    TotalScore DECIMAL(18,2)
)

-- CTE (Common Table Expression)
;WITH RankedPlayers AS (
    SELECT
        PlayerId,
        Score,
        ROW_NUMBER() OVER (PARTITION BY TeamId ORDER BY Score DESC) AS Rank
    FROM Players
)
SELECT * FROM RankedPlayers WHERE Rank = 1
```

#### Error Handling
```sql
BEGIN TRY
    -- Operations that might fail
END TRY
BEGIN CATCH
    SELECT
        ERROR_NUMBER() AS ErrorNumber,
        ERROR_MESSAGE() AS ErrorMessage,
        ERROR_SEVERITY() AS ErrorSeverity,
        ERROR_STATE() AS ErrorState,
        ERROR_PROCEDURE() AS ErrorProcedure,
        ERROR_LINE() AS ErrorLine

    -- Custom error
    RAISERROR('Custom error message', 16, 1)
    -- Or use THROW (SQL 2012+)
    THROW 50001, 'Custom error message', 1
END CATCH
```

## Reading Stored Procedures

### Identify Key Elements

1. **Parameters**: Input/Output params define the contract
2. **Return values**: `RETURN @Code` for status codes
3. **Result sets**: `SELECT` statements that return data
4. **Side effects**: `INSERT`, `UPDATE`, `DELETE` operations
5. **Transactions**: Scope of atomicity

### Business Logic Extraction

Look for:
- **Validation rules**: `IF` conditions that reject input
- **Calculations**: Math operations, aggregations
- **State transitions**: Status field updates
- **Audit trails**: Logging inserts
- **External calls**: Linked servers, dynamic SQL

### Dependency Analysis

```sql
-- Find procedure dependencies
SELECT DISTINCT
    referenced_schema_name,
    referenced_entity_name,
    referenced_class_desc
FROM sys.dm_sql_referenced_entities('Schema.ProcedureName', 'OBJECT')

-- Find what calls this procedure
SELECT DISTINCT
    referencing_schema_name,
    referencing_entity_name
FROM sys.dm_sql_referencing_entities('Schema.ProcedureName', 'OBJECT')
```

## Common SQL Server Functions

### String Functions
| Function | Purpose | Example |
|----------|---------|---------|
| `LEN()` | String length | `LEN(@PlayerName)` |
| `TRIM()` | Remove whitespace | `TRIM(@Value)` |
| `SUBSTRING()` | Extract part | `SUBSTRING(@Str, 1, 10)` |
| `CONCAT()` | Join strings | `CONCAT(@First, ' ', @Last)` |
| `REPLACE()` | Replace text | `REPLACE(@Str, 'old', 'new')` |

### Date Functions
| Function | Purpose | Example |
|----------|---------|---------|
| `GETDATE()` | Current datetime | `GETDATE()` |
| `GETUTCDATE()` | Current UTC | `GETUTCDATE()` |
| `DATEADD()` | Add interval | `DATEADD(DAY, 7, @Date)` |
| `DATEDIFF()` | Difference | `DATEDIFF(DAY, @Start, @End)` |
| `FORMAT()` | Format date | `FORMAT(@Date, 'yyyy-MM-dd')` |

### Aggregate Functions
| Function | Purpose |
|----------|---------|
| `SUM()` | Total |
| `COUNT()` | Count rows |
| `AVG()` | Average |
| `MAX()` / `MIN()` | Extremes |
| `STRING_AGG()` | Concatenate (2017+) |

### Window Functions
```sql
-- Row numbering
ROW_NUMBER() OVER (PARTITION BY TeamId ORDER BY Score DESC)

-- Running totals
SUM(Points) OVER (PARTITION BY PlayerId ORDER BY MatchDate)

-- Ranking
RANK() OVER (ORDER BY Score DESC)
DENSE_RANK() OVER (ORDER BY Score DESC)

-- Lead/Lag (previous/next row values)
LAG(Score, 1) OVER (ORDER BY MatchDate) AS PreviousScore
LEAD(Score, 1) OVER (ORDER BY MatchDate) AS NextScore
```

## Best Practices

### Performance

- **Use SET NOCOUNT ON** - Reduces network traffic
- **Avoid SELECT *** - Specify only needed columns
- **Use EXISTS over COUNT** for existence checks
- **Prefer set-based operations over cursors**
- **Index columns used in WHERE and JOIN**

```sql
-- Bad: Cursor for updates
DECLARE cursor CURSOR FOR SELECT Id FROM Players
-- Process one by one...

-- Good: Set-based update
UPDATE Players
SET Status = 'Inactive'
WHERE LastLogin < DATEADD(MONTH, -6, GETDATE())
```

### Security

- **Use parameterized queries** - Prevent SQL injection
- **Avoid dynamic SQL when possible**
- **Use schema-qualified names** - `Gaming.Players` not just `Players`
- **Grant minimal permissions** - Execute only, not table access

```sql
-- Bad: Dynamic SQL without parameterization
SET @SQL = 'SELECT * FROM Players WHERE Name = ''' + @Name + ''''

-- Good: Parameterized dynamic SQL
SET @SQL = 'SELECT * FROM Players WHERE Name = @PlayerName'
EXEC sp_executesql @SQL, N'@PlayerName NVARCHAR(100)', @Name
```

### Maintainability

- **Use meaningful names** - Prefix with action: `usp_GetPlayerStats`
- **Document complex logic** - Header comments with purpose
- **Keep procedures focused** - Single responsibility
- **Handle NULL explicitly** - Use ISNULL or COALESCE

```sql
-- Procedure naming convention
CREATE PROCEDURE Gaming.usp_GetPlayerStats
    @PlayerId INT
AS
BEGIN
    /*
    Purpose: Retrieve player statistics summary
    Author: [Author]
    Date: [Date]
    */

    SELECT
        p.PlayerId,
        p.PlayerName,
        ISNULL(SUM(gs.Score), 0) AS TotalScore,
        COUNT(gs.SessionId) AS GamesPlayed
    FROM Gaming.Players p
    LEFT JOIN Gaming.GameSessions gs ON p.PlayerId = gs.PlayerId
    WHERE p.PlayerId = @PlayerId
    GROUP BY p.PlayerId, p.PlayerName
END
```

### Transaction Management

```sql
-- Proper transaction handling
BEGIN TRY
    BEGIN TRANSACTION;

    UPDATE Gaming.Players SET Coins = Coins - @Cost WHERE PlayerId = @PlayerId;
    INSERT INTO Gaming.Purchases (PlayerId, ItemId, PurchaseDate) VALUES (@PlayerId, @ItemId, GETDATE());

    COMMIT TRANSACTION;
END TRY
BEGIN CATCH
    IF @@TRANCOUNT > 0
        ROLLBACK TRANSACTION;

    -- Log error details
    INSERT INTO Gaming.ErrorLog (ErrorMessage, ErrorDate)
    VALUES (ERROR_MESSAGE(), GETDATE());

    THROW;
END CATCH
```

## Anti-Patterns to Avoid

- **Dynamic SQL without parameterization** - SQL injection risk
- **Cursors when set-based operations work** - Performance killer
- **SELECT * in production code** - Fragile and wasteful
- **Missing indexes on WHERE/JOIN columns** - Slow queries
- **Transactions held open during external calls** - Blocking
- **Implicit conversions** - Hidden performance issues
- **NOT IN with NULL values** - Unexpected empty results

```sql
-- Anti-pattern: NOT IN with potential NULLs
SELECT * FROM Players WHERE TeamId NOT IN (SELECT TeamId FROM InactiveTeams)

-- Better: NOT EXISTS handles NULLs correctly
SELECT * FROM Players p
WHERE NOT EXISTS (SELECT 1 FROM InactiveTeams it WHERE it.TeamId = p.TeamId)
```
