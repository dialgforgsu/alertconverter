# SQL File Auto-growth Exceeds Free Space

## Overview
Detects when the next auto-growth event would exceed either available free disk space or the configured maximum file size. When the next auto-growth cannot complete, the database will go into a read-only or suspect state.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
-- Uses SQL Sentry internal tables (PerformanceAnalysisSqlFile, PerformanceAnalysisDeviceLogicalDisk)
-- Calculates space debt: free disk space minus next growth size
```

> **Note:** Uses SQL Sentry's internal monitoring database tables with pre-collected disk and file size data.

---

## Redgate Monitor Custom Metric

### Metric Name
Files Where Next Autogrowth May Fail

### Description
Returns the count of database files where the configured auto-growth increment exceeds 50% of the remaining disk free space (approximated using volume stats). This is an early warning that the next auto-growth event may fail.

> **Caveat:** Exact disk free space per volume requires `sys.dm_os_volume_stats` (SQL Server 2008 R2+). This query provides a file-level count of at-risk files.

### Target Level
Instance

### T-SQL Query
```sql
SELECT COUNT(*) AS FilesAtRiskOfGrowthFailure
FROM sys.master_files AS mf
CROSS APPLY sys.dm_os_volume_stats(mf.database_id, mf.file_id) AS vs
WHERE mf.growth > 0
  AND mf.state_desc = N'ONLINE'
  AND (
    -- growth is in pages (if not percentage); check if next growth > 50% of available space
    (mf.is_percent_growth = 0 AND (CAST(mf.growth AS BIGINT) * 8192) > (vs.available_bytes * 0.5))
    OR
    -- max_size check: if max_size is set and current size + growth > max_size
    (mf.max_size > 0 AND (mf.size + mf.growth) > mf.max_size)
  );
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 15 minutes |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested)
| Severity | Threshold |
|---|---|
| High | > 0 |

### Notes
- `sys.dm_os_volume_stats` requires at minimum VIEW SERVER STATE permission.
- Pre-grow files proactively during maintenance windows rather than relying on auto-growth.
- Redgate Monitor's built-in disk usage projections show remaining space and estimated time-to-full per volume.
