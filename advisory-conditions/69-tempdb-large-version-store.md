# Tempdb Large Version Store

## Overview
Checks for a TempDB version store that is greater than 100,000 KB (approximately 100 MB). A large version store indicates long-running snapshot isolation transactions are preventing version store cleanup.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
No T-SQL standalone query â SQL Sentry monitors the version store size via its internal collection. The threshold is > 100,000 KB.

---

## Redgate Monitor Custom Metric

### Metric Name
TempDB Version Store Size (MB)

### Description
Returns the current TempDB version store size in MB. A large version store is caused by long-running transactions under snapshot isolation (RCSI or SI) that prevent old row versions from being cleaned up.

### Target Level
Instance

### T-SQL Query
```sql
SELECT CAST(SUM(version_store_reserved_page_count) * 8.0 / 1024.0 AS DECIMAL(18,2)) AS VersionStoreSizeMB
FROM sys.dm_db_file_space_usage
WHERE database_id = 2;
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 5 minutes |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested)
| Severity | Threshold |
|---|---|
| High | > 1024 MB |
| Medium | > 512 MB |
| Low | > 100 MB |

### Notes
- The version store grows when snapshot isolation (RCSI or SI) is enabled and long-running transactions prevent cleanup.
- Identify the long-running snapshot transaction: `SELECT * FROM sys.dm_tran_active_snapshot_database_transactions ORDER BY elapsed_time_seconds DESC;`
- Killing the oldest snapshot transaction will allow the version store to be cleaned up by the cleanup background task.
