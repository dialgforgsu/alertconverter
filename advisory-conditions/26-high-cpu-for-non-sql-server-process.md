# High CPU for Non-SQL Server Process

## Overview
On dedicated SQL Servers, most CPU should relate to `sqlservr.exe`. This condition detects when CPU is high and at least 25% is consumed by non-SQL Server processes, indicating resource contention.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
No standalone T-SQL â SQL Sentry uses Windows per-process CPU tracking via WMI to compare SQL Server process CPU against total CPU utilisation.

---

## Redgate Monitor Custom Metric

### Metric Name
Non-SQL Server CPU Percentage (Estimated)

### Description
Estimates CPU consumed by processes other than SQL Server by subtracting SQL Server process CPU from total system CPU using ring buffer data. A high value during overall high CPU indicates other processes are competing with SQL Server.

> **Caveat:** This is an approximation. Redgate Monitor's Activity Graph â Windows Processes tab shows definitive per-process CPU breakdown.

### Target Level
Instance

### T-SQL Query
```sql
SELECT TOP 1
    (100
     - record.value('(./Record/SchedulerMonitorEvent/SystemHealth/SystemIdle)[1]', 'INT')
     - record.value('(./Record/SchedulerMonitorEvent/SystemHealth/ProcessUtilization)[1]', 'INT')
    ) AS NonSqlCpuPercent
FROM (
    SELECT CAST(record AS XML) AS record
    FROM sys.dm_os_ring_buffers
    WHERE ring_buffer_type = N'RING_BUFFER_SCHEDULER_MONITOR'
      AND record LIKE N'%<SystemHealth>%'
) AS rb
ORDER BY record.value('(./Record/@id)[1]', 'BIGINT') DESC;
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
| High | > 40 |
| Medium | > 25 |
| Low | > 15 |

### Notes
- Result can be negative if ring buffer data is stale â add a CASE WHEN result < 0 THEN 0 guard in production.
- Common culprits on a dedicated SQL Server: antivirus scanning data directories, backup agents, monitoring tools, SSRS/SSAS co-located on the same host.
- Redgate Monitor's Windows Processes tab in Performance Analysis provides the definitive per-process breakdown.
