# Foreign Keys Not Trusted

## Overview
Evaluates to true when `sys.foreign_keys.is_not_trusted = 1` on enabled, non-replication foreign keys. Untrusted FKs prevent the query optimiser from using them for join elimination and constraint-based simplifications.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
DECLARE @sql nvarchar(max) = N'';
SELECT @sql = @sql + N'UNION ALL
  SELECT DBName = N''' + name + ''' COLLATE Latin1_General_BIN,
  FKsNotTrusted = (SELECT COUNT(*) FROM ' + QUOTENAME(name) + '.sys.foreign_keys
      WHERE is_not_trusted = 1 AND is_not_for_replication = 0 AND is_disabled = 0)'
FROM sys.databases WHERE database_id > 4 AND state = 0;
SET @sql = N'SELECT DBName, FKsNotTrusted FROM (' + STUFF(@sql,1,10,N'') + N') AS x WHERE FKsNotTrusted > 0;';
EXEC sys.sp_executesql @sql;
```

---

## Redgate Monitor Custom Metric

### Metric Name
Untrusted Foreign Keys (Instance Total)

### Description
Returns the total count of enabled, non-replication foreign keys across all online user databases that are not trusted. A non-zero value means the query optimiser cannot use these FKs for plan simplification.

### Target Level
Instance

### T-SQL Query
```sql
DECLARE @sql NVARCHAR(MAX) = N'';
SELECT @sql = @sql + N'
SELECT COUNT(*) FROM ' + QUOTENAME(name) + N'.sys.foreign_keys
WHERE is_not_trusted = 1 AND is_not_for_replication = 0 AND is_disabled = 0;'
FROM sys.databases WHERE database_id > 4 AND state_desc = N'ONLINE' AND is_read_only = 0;

CREATE TABLE #fk (cnt INT);
INSERT INTO #fk EXEC sys.sp_executesql @sql;
SELECT SUM(cnt) AS UntrustedForeignKeys FROM #fk;
DROP TABLE #fk;
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
| High | > 20 |
| Medium | > 5 |
| Low | > 0 |

### Notes
- Fix with: `ALTER TABLE [schema].[table] WITH CHECK CHECK CONSTRAINT [fk_name];`
- Common after bulk loads where constraints were disabled for performance. Restore trust as part of your ETL post-processing.
