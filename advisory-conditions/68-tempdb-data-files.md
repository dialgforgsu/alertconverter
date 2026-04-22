# Tempdb Data Files

## Overview
Evaluates to true when the number of TempDB data files is suboptimal. Microsoft recommends: if logical processors <= 8, use the same number of data files as logical processors. If > 8, use 8 files and increase by multiples of 4 if contention continues.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
DECLARE @TempDBFiles TinyInt, @CPUCount TinyInt
SELECT @CPUCount = COUNT(*) FROM sys.dm_os_schedulers WHERE STATUS = N'VISIBLE ONLINE' AND IS_ONLINE = 1;
SELECT @TempDBFiles = COUNT(*) FROM master.sys.master_files WHERE database_id = 2 AND file_id <> 2;
SELECT CASE WHEN @TempDBFiles = @CPUCount THEN 0
    WHEN @TempDBFiles > @CPUCount THEN 1
    WHEN @CPUCount >= 4 AND @TempDBFiles < 4 THEN 2
    ELSE 0 END [Contention];
```

---

## Redgate Monitor Custom Metric

### Metric Name
TempDB Data File vs CPU Ratio

### Description
Returns the difference between the recommended number of TempDB data files and the actual count. A positive value means too few files; negative means too many (also suboptimal). Zero is optimal.

### Target Level
Instance

### T-SQL Query
```sql
DECLARE @cpuCount INT, @tempdbFiles INT, @recommended INT;

SELECT @cpuCount = COUNT(*)
FROM sys.dm_os_schedulers
WHERE [status] = N'VISIBLE ONLINE' AND is_online = 1;

SELECT @tempdbFiles = COUNT(*)
FROM sys.master_files
WHERE database_id = 2 AND type_desc = N'ROWS';

SET @recommended = CASE
    WHEN @cpuCount <= 8 THEN @cpuCount
    ELSE 8
END;

SELECT (@recommended - @tempdbFiles) AS TempdbFileDeficit;
-- Positive = too few files, 0 = optimal, Negative = too many files
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
| High | > 4 (significantly too few files) |
| Medium | > 0 (fewer files than recommended) |

### Notes
- Equal-sized TempDB data files and trace flag 1118 (SQL Server 2016 default) are the key requirements for reducing SGAM/GAM contention.
- In SQL Server 2016+, TF 1117 and TF 1118 are on by default for TempDB â no need to manually enable them.
- After adding TempDB files, all files must be sized equally to ensure round-robin allocation works correctly.
