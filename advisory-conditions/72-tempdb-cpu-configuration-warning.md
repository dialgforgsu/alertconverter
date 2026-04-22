# Tempdb/CPU Configuration Warning

## Overview
Checks that you have one TempDB data file per logical CPU core (up to 8). The first query counts visible online CPUs; the second counts TempDB data files. Alerts when the counts don't match (for systems with <= 8 cores).

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT count(cpu_id) AS cpu_count FROM sys.dm_os_schedulers WHERE status = 'VISIBLE ONLINE';
SELECT count(Name) AS tempdb_files FROM sys.master_files WHERE database_id = 2 AND type_desc <> 'LOG';
```

---

## Redgate Monitor Custom Metric

### Metric Name
TempDB Files vs CPU Count Delta (<=8 cores)

### Description
Returns the difference between the CPU count and the TempDB data file count on servers with 8 or fewer logical cores. A non-zero value indicates the TempDB file count does not match the CPU count, which may cause allocation contention.

### Target Level
Instance

### T-SQL Query
```sql
DECLARE @cpuCount INT, @tempdbFiles INT;

SELECT @cpuCount = COUNT(*)
FROM sys.dm_os_schedulers
WHERE [status] = N'VISIBLE ONLINE' AND is_online = 1;

SELECT @tempdbFiles = COUNT(*)
FROM sys.master_files
WHERE database_id = 2 AND type_desc = N'ROWS';

-- Only flag when CPU count <= 8 (handled separately for > 8 in condition 73)
SELECT CASE WHEN @cpuCount <= 8 THEN ABS(@cpuCount - @tempdbFiles) ELSE 0 END
    AS TempdbCpuFileDelta;
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
| High | > 2 |
| Medium | > 0 |

### Notes
- This condition applies to servers with <= 8 logical cores. See condition 73 for the > 8 core variant.
- The goal is 1 TempDB data file per logical core on small servers to eliminate allocation bitmap contention.
- All TempDB data files must be equal-sized for round-robin allocation to distribute evenly.
