# Check Error Log Size

## Overview
Checks the size of the SQL Server error log. Used as a gate inside other conditions (e.g. High Number of Failed Logins) to prevent slow queries against very large log files.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
DECLARE @currfilesize int
CREATE TABLE #err_log_tmp(ArchiveNo int, CreateDate nvarchar(128), Size int)
INSERT #err_log_tmp EXEC master.dbo.sp_enumerrorlogs
SELECT TOP 1 @currfilesize = er.Size FROM #err_log_tmp er ORDER BY [ArchiveNo] ASC
DROP TABLE #err_log_tmp
SELECT @currfilesize;
```

---

## Redgate Monitor Custom Metric

### Metric Name
SQL Server Error Log Size (MB)

### Description
Returns the size in MB of the current active SQL Server error log. A large error log slows log-scanning queries and may indicate excessive error activity.

### Target Level
Instance

### T-SQL Query
```sql
CREATE TABLE #err_log_size (ArchiveNo INT, CreateDate NVARCHAR(128), Size INT);
INSERT INTO #err_log_size EXEC master.dbo.sp_enumerrorlogs;
SELECT TOP 1 CAST(Size AS DECIMAL(18,2)) / 1024.0 / 1024.0 AS ErrorLogSizeMB
FROM #err_log_size ORDER BY ArchiveNo ASC;
DROP TABLE #err_log_size;
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 30 minutes |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested)
| Severity | Threshold |
|---|---|
| High | > 500 MB |
| Medium | > 100 MB |
| Low | > 50 MB |

### Notes
- Cycle the log with `EXEC sp_cycle_errorlog;` to keep it manageable.
- Configure max error log file count via SSMS: right-click SQL Server Logs > Configure.
- A large log is often a symptom of excessive errors â investigate the root cause.
