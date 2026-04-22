# High CPU

## Overview
Sustained CPU utilisation greater than 90% may indicate a CPU bottleneck. On dedicated SQL Server machines, most CPU should be associated with the SQL Server process. Use Top Queries and Performance Analysis to identify consumers.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
No T-SQL â SQL Sentry reads system-wide CPU % via Windows WMI/PerfMon. Alerts when sustained CPU > 90%.

---

## Redgate Monitor Custom Metric

### Metric Name
SQL Server Process CPU Utilisation (%)

### Description
Returns the SQL Server process CPU utilisation % from the ring buffer. Supplements Redgate Monitor's built-in CPU alert with the SQL Server process-specific component, enabling separation of SQL Server CPU from other processes.

> **Note:** Redgate Monitor has a built-in High CPU alert covering system-wide CPU. This metric provides SQL Server process-specific CPU %.

### Target Level
Instance

### T-SQL Query
```sql
SELECT TOP 1
    record.value('(./Record/SchedulerMonitorEvent/SystemHealth/ProcessUtilization)[1]', 'INT')
        AS SqlServerCpuPercent
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
| High | > 90 |
| Medium | > 75 |
| Low | > 60 |

### Notes
- The ring buffer stores the last ~256 scheduler monitor records.
- For identifying which queries drive high CPU, use Redgate Monitor's Top Queries view sorted by CPU.
- Redgate Monitor's built-in CPU alert is the primary mechanism â this custom metric provides SQL Server process granularity.
