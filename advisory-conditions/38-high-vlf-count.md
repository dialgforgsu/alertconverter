# High VLF Count

## Overview
Too many Virtual Log Files (VLFs) in transaction logs can lead to increased backup and recovery times, slow log operations, and performance problems. VLF count is driven by initial log size and auto-growth increments.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
SQL Sentry populates VLF counts via internal `DBCC LOGINFO` execution. No standalone T-SQL exposed in the advisory conditions UI.

---

## Redgate Monitor Custom Metric

### Metric Name
Max VLF Count (Any Database)

### Description
Returns the maximum VLF count across all online databases. Requires SQL Server 2016+ where `sys.dm_db_log_info` is available. High VLF counts (> 1,000) can significantly slow database recovery, log backups, and log maintenance.

### Target Level
Instance

### T-SQL Query
```sql
DECLARE @sql NVARCHAR(MAX) = N'';
SELECT @sql = @sql + N'
UNION ALL SELECT COUNT(*) AS vlf_count
FROM sys.dm_db_log_info(' + CAST(database_id AS NVARCHAR(10)) + N')'
FROM sys.databases
WHERE state_desc = N'ONLINE';

SET @sql = STUFF(@sql, 1, 12, N'');

CREATE TABLE #vlf (vlf_count INT);
INSERT INTO #vlf EXEC sys.sp_executesql @sql;
SELECT MAX(vlf_count) AS MaxVLFCount FROM #vlf;
DROP TABLE #vlf;
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
| High | > 1000 |
| Medium | > 500 |
| Low | > 200 |

### Notes
- `sys.dm_db_log_info` requires SQL Server 2016+. For earlier versions use `DBCC LOGINFO` in a cursor.
- To fix excessive VLFs: grow the log to its desired final size in a single operation, then shrink and regrow to consolidate VLFs.
- Redgate Monitor's built-in disk analysis page shows VLF counts per database.
