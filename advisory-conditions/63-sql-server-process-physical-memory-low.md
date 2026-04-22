# SQL Server Process Physical Memory Low

## Overview
Evaluates to true when `sys.dm_os_process_memory.process_physical_memory_low` is true. This indicates the SQL Server process is responding to a low physical memory notification from Windows.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT process_physical_memory_low
FROM sys.dm_os_process_memory WITH (NOLOCK);
```

---

## Redgate Monitor Custom Metric

### Metric Name
SQL Server Process Physical Memory Low Flag

### Description
Returns 1 when SQL Server has received a low physical memory notification from Windows and is actively responding (e.g. shrinking the buffer pool). A value of 1 indicates the server is under physical memory pressure.

### Target Level
Instance

### T-SQL Query
```sql
SELECT CAST(process_physical_memory_low AS INT) AS ProcessPhysicalMemoryLow
FROM sys.dm_os_process_memory WITH (NOLOCK);
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 1 minute |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested)
| Severity | Threshold |
|---|---|
| High | > 0 (i.e., = 1) |

### Notes
- When this flag is 1, SQL Server is actively shrinking the buffer pool to free physical memory.
- Pair with Page Life Expectancy (condition 45) and Available Memory (condition 44) to build a complete memory pressure picture.
- Common causes: `max server memory` set too high leaving insufficient memory for the OS, memory leak in a linked server or CLR component, or genuine server under-provisioning.
