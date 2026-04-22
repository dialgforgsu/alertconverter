# SQL Server Process Virtual Memory Low

## Overview
Evaluates to true when `sys.dm_os_process_memory.process_virtual_memory_low` is true. This indicates the SQL Server process needs more virtual address space, which can occur on 32-bit systems or when virtual memory is heavily fragmented.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT process_virtual_memory_low
FROM sys.dm_os_process_memory WITH (NOLOCK);
```

---

## Redgate Monitor Custom Metric

### Metric Name
SQL Server Process Virtual Memory Low Flag

### Description
Returns 1 when SQL Server has received a low virtual memory notification. On 64-bit systems this is very rare â a value of 1 is a serious indicator of virtual address space exhaustion, often caused by excessive memory allocations outside the buffer pool (CLR, linked servers, SSRS, etc.).

### Target Level
Instance

### T-SQL Query
```sql
SELECT CAST(process_virtual_memory_low AS INT) AS ProcessVirtualMemoryLow
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
- On 64-bit SQL Server, virtual memory low is extremely rare and indicates a serious problem.
- Check `sys.dm_os_virtual_address_dump` for virtual address space fragmentation.
- Common causes on 64-bit: excessive `max server memory`, large numbers of in-memory OLTP objects, or very large sort operations requesting contiguous virtual memory.
