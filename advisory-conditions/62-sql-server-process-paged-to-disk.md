# SQL Server Process Paged to Disk

## Overview
Under severe memory pressure, Windows can page the SQL Server process to disk, dramatically impacting performance. When SQL Server's working set (physical RAM in use) drops far below its committed virtual memory, portions of the process have been paged to disk.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
SQL Sentry reads the ring buffer for the latest SQL Server process working set utilisation value.

---

## Redgate Monitor Custom Metric

### Metric Name
SQL Server Process Working Set Utilisation (%)

### Description
Returns the SQL Server process working set utilisation percentage — the ratio of physical memory currently in use to total committed virtual memory. When Windows pages SQL Server to disk, physical memory in use drops well below committed virtual memory. Values below 75% indicate active paging and severe performance degradation.

### Target Level
Instance

### T-SQL Query
```sql
SELECT CAST(
    100.0 * physical_memory_in_use_kb
    / NULLIF(virtual_memory_committed_kb, 0)
AS DECIMAL(10,2)) AS WorkingSetUtilisationPct
FROM sys.dm_os_process_memory;
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
| High | < 50 (%) |
| Medium | < 75 (%) |

### Notes
- Alert direction is **below** threshold.
- `physical_memory_in_use_kb` is the SQL Server process working set — pages currently backed by physical RAM. `virtual_memory_committed_kb` is total committed virtual memory. When the ratio drops, pages have been swapped to disk by Windows.
- Assigning the "Lock Pages in Memory" (LPIM) privilege to the SQL Server service account prevents Windows from paging the buffer pool, but does not prevent paging of thread stacks.
- Supplement with `page_fault_count` from `sys.dm_os_process_memory` — a rapidly increasing count confirms active paging to disk.
- If this alert fires, the server is in critical memory distress — immediate action required.