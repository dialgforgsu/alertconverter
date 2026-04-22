# Tempdb Unequal File Size

## Overview
The benefits of multiple TempDB files are lost if one file grows larger than the others, breaking the round-robin allocation that prevents SGAM/GAM contention. This condition evaluates to true if TempDB data files are not all the same size.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT count(distinct size) AS unique_sizes
FROM sys.master_files
WHERE database_id = 2 AND type_desc <> 'LOG';
```

---

## Redgate Monitor Custom Metric

### Metric Name
TempDB Data File Unique Size Count

### Description
Returns the count of distinct file sizes among TempDB data files. A value of 1 means all files are equal-sized (optimal). Any value > 1 means files have grown unevenly, which breaks round-robin allocation and can cause contention.

### Target Level
Instance

### T-SQL Query
```sql
SELECT COUNT(DISTINCT size) AS TempdbUniqueFileSizeCount
FROM sys.master_files
WHERE database_id = 2
  AND type_desc = N'ROWS';
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
| High | > 1 |

### Notes
- To equalise: shrink the larger files to match the smallest, then resize all files to the same large pre-allocated size.
- Set `FILEGROWTH` to the same value on all TempDB data files to prevent them growing unevenly in the future.
- In SQL Server 2016+, TF 1117 is active by default for TempDB, causing all data files to grow simultaneously when any one needs to grow â this prevents unequal sizes from auto-growth.
