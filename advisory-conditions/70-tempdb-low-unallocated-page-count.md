# Tempdb Low Unallocated Page Count

## Overview
Evaluates to true when less than 10% of TempDB's total page count is unallocated. This may indicate TempDB is running out of space and will soon auto-grow or hit a size limit.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT (CONVERT(DECIMAL(18,0),(SUM(unallocated_extent_page_count)))/(SUM(total_page_count))*100) AS percent_used
FROM tempdb.sys.dm_db_file_space_usage;
```

---

## Redgate Monitor Custom Metric

### Metric Name
TempDB Unallocated Space Percentage

### Description
Returns the percentage of TempDB's total allocated space that is currently unallocated (free). Values below 10% indicate TempDB is nearly full and auto-growth or a size limit may be imminent.

### Target Level
Instance

### T-SQL Query
```sql
SELECT CAST(
    100.0 * SUM(unallocated_extent_page_count)
    / NULLIF(SUM(total_page_count), 0)
AS DECIMAL(10,2)) AS TempdbUnallocatedPct
FROM tempdb.sys.dm_db_file_space_usage;
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
| High | < 5% |
| Medium | < 10% |
| Low | < 20% |

### Notes
- Alert direction is **below** threshold.
- A sudden drop in TempDB free space often correlates with a runaway sort, hash join, or version store growth event.
- Pre-size TempDB data files large enough to avoid frequent auto-growth â auto-growth of TempDB causes all data files to grow simultaneously if they are equal-sized.
