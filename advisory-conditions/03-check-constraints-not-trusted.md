# Check Constraints Not Trusted

## Overview
Evaluates to true when `sys.check_constraints.is_not_trusted = 1` on enabled, non-replication constraints. Untrusted check constraints cannot be used by the query optimiser for plan elimination, reducing performance.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
DECLARE @sql nvarchar(max) = N'';
SELECT @sql = @sql + N'UNION ALL
  SELECT DBName = N''' + name + ''' COLLATE Latin1_General_BIN,
  CCsNotTrusted = (SELECT COUNT(*) FROM ' + QUOTENAME(name) + '.sys.check_constraints
      WHERE is_not_trusted = 1 AND is_not_for_replication = 0 AND is_disabled = 0)'
FROM sys.databases WHERE database_id > 4 AND state = 0;
SET @sql = N'SELECT DBName, CCsNotTrusted FROM (' + STUFF(@sql,1,10,N'') + N') AS x WHERE CCsNotTrusted > 0;';
EXEC sys.sp_executesql @sql;
```

---

## Redgate Monitor Custom Metric

### Metric Name
Untrusted Check Constraints (Instance Total)

### Description
Returns the total count of enabled, non-replication check constraints across all online user databases marked as not trusted. A non-zero value means the query optimiser cannot use these constraints for plan optimisation.

### Target Level
Instance

### T-SQL Query
```sql
DECLARE @sql NVARCHAR(MAX) = N'';
SELECT @sql = @sql + N'
SELECT COUNT(*) FROM ' + QUOTENAME(name) + N'.sys.check_constraints
WHERE is_not_trusted = 1 AND is_not_for_replication = 0 AND is_disabled = 0;'
FROM sys.databases WHERE database_id > 4 AND state_desc = N'ONLINE' AND is_read_only = 0;

CREATE TABLE #c (cnt INT);
INSERT INTO #c EXEC sys.sp_executesql @sql;
SELECT SUM(cnt) AS UntrustedCheckConstraints FROM #c;
DROP TABLE #c;
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 60 minutes |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested)
| Severity | Threshold |
|---|---|
| High | > 10 |
| Medium | > 5 |
| Low | > 0 |

### Notes
- Fix with: `ALTER TABLE [schema].[table] WITH CHECK CHECK CONSTRAINT [constraint_name];`
- Always use `WITH CHECK CHECK CONSTRAINT` â `WITH NOCHECK` re-enables enforcement but does not restore trust.
