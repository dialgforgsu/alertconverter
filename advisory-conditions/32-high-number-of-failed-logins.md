# High Number of Failed Logins

## Overview
Checks for more than 10 failed logins in the last 2 minutes. A high count may indicate an unauthorised access attempt. Requires SQL Server login auditing to be set to "Failed logins only" or "Both failed and successful logins."

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
-- Step 1: guard on log size
DECLARE @currfilesize int
CREATE TABLE #err_log_tmp(ArchiveNo int, CreateDate nvarchar(128), Size int)
INSERT #err_log_tmp EXEC master.dbo.sp_enumerrorlogs
SELECT TOP 1 @currfilesize = er.Size FROM #err_log_tmp er ORDER BY [ArchiveNo] ASC
DROP TABLE #err_log_tmp
SELECT @currfilesize;

-- Step 2: count failed logins
CREATE TABLE #log (logdate DATETIME, info VARCHAR(25), data VARCHAR(200));
INSERT INTO #log EXECUTE sp_readerrorlog 0, 1, 'Login failed';
SELECT count(*) AS occurences FROM #log WHERE logdate > dateadd(minute, -2, getdate());
DROP TABLE #log;
```

---

## Redgate Monitor Custom Metric

### Metric Name
Failed Logins (Last 2 Minutes)

### Description
Returns the count of failed login attempts in the SQL Server error log within the last 2 minutes. Returns -1 if the error log is too large to scan safely (> 5 MB). Requires login failure auditing to be enabled.

### Target Level
Instance

### T-SQL Query
```sql
DECLARE @logSizeBytes INT = 0;
CREATE TABLE #log_size (ArchiveNo INT, CreateDate NVARCHAR(128), Size INT);
INSERT INTO #log_size EXEC master.dbo.sp_enumerrorlogs;
SELECT TOP 1 @logSizeBytes = Size FROM #log_size ORDER BY ArchiveNo ASC;
DROP TABLE #log_size;

IF @logSizeBytes > 5242880
BEGIN SELECT -1 AS FailedLoginCount; RETURN; END;

CREATE TABLE #fl (logdate DATETIME, info VARCHAR(25), data VARCHAR(200));
INSERT INTO #fl EXEC sp_readerrorlog 0, 1, N'Login failed';
SELECT COUNT(*) AS FailedLoginCount FROM #fl
WHERE logdate > DATEADD(MINUTE, -2, GETDATE());
DROP TABLE #fl;
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 2 minutes |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested)
| Severity | Threshold |
|---|---|
| High | > 50 |
| Medium | > 20 |
| Low | > 10 |

### Notes
- A return of -1 means the error log is too large to scan â cycle it with `EXEC sp_cycle_errorlog;`.
- Enable login auditing: SSMS â Server Properties â Security â Login Auditing.
- For high-volume environments, consider SQL Server Audit or Extended Events for scalable failed login detection.
