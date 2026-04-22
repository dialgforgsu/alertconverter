# Tempdb/CPU Configuration Warning > 8 CPUs

## Overview
For servers with more than 8 CPUs, start with 8 TempDB data files and add multiples of 4 if contention continues. Microsoft recommends at least a quarter of the core count as TempDB files on high-core-count servers. Alerts when TempDB file count is below this minimum.

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
TempDB Files Deficit vs CPU Quarter (>8 cores)

### Description
For servers with > 8 logical cores, returns the deficit between the minimum recommended TempDB file count (max(8, CPU_count / 4)) and the actual file count. A positive value means TempDB is under-provisioned relative to CPU count.

### Target Level
Instance

### T-SQL Query
```sql
DECLARE @cpuCount INT, @tempdbFiles INT, @minRecommended INT, @deficit INT;

SELECT @cpuCount = COUNT(*)
FROM sys.dm_os_schedulers
WHERE [status] = N'VISIBLE ONLINE' AND is_online = 1;

SELECT @tempdbFiles = COUNT(*)
FROM sys.master_files
WHERE database_id = 2 AND type_desc = N'ROWS';

SET @minRecommended = CASE
    WHEN @cpuCount <= 8 THEN 0  -- handled by condition 72
    WHEN @cpuCount / 4 > 8 THEN @cpuCount / 4
    ELSE 8
END;

SET @deficit = @minRecommended - @tempdbFiles;

SELECT CASE WHEN @cpuCount > 8 AND @deficit > 0 THEN @deficit ELSE 0 END
    AS TempdbFileDeficit;
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
| High | > 4 |
| Medium | > 0 |

### Notes
- Compatible with SQL Server 2016 and later. `GREATEST()` (SQL Server 2022+ only) has been replaced with a `CASE` expression for broader compatibility.
- Microsoft's rule of thumb for high-core-count servers: start with 8 TempDB data files, monitor for `PAGELATCH_EX`/`PAGELATCH_SH` waits on TempDB allocation pages, and add files in multiples of 4 if contention persists.
- All TempDB data files must remain equal-sized for round-robin allocation to distribute evenly.