# High Context Switches

## Overview
Context switches represent the rate at which processors switch between threads. Consistently high values over 7,500 per logical processor indicate the server is spending too much time switching threads rather than running them productively.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
No T-SQL â SQL Sentry reads `System: Context Switches/sec` via Windows WMI/PerfMon. Alerts when value > 7,500 per logical processor.

---

## Redgate Monitor Custom Metric

### Metric Name
Average Runnable Tasks Count (Context Switch Proxy)

### Description
The Windows `System: Context Switches/sec` counter is not accessible via T-SQL. This metric uses the SQL Server runnable tasks count as a proxy â when the host is CPU-starved, SQL Server's runnable queue grows as threads wait for CPU time.

> **Caveat:** Redgate Monitor's WMI collection surfaces `System: Context Switches/sec` natively in the Activity Graph (OS category). Use this custom metric as a supplementary alert; use the Activity Graph counter for the definitive measurement.

### Target Level
Instance

### T-SQL Query
```sql
SELECT AVG(runnable_tasks_count) AS AvgRunnableTasksCount
FROM sys.dm_os_schedulers WITH (NOLOCK)
WHERE [status] = N'VISIBLE ONLINE';
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
| High | > 10 |
| Medium | > 3 |

### Notes
- Check Redgate Monitor's Activity Graph â OS category for the `Context Switches/sec` Windows counter.
- High context switches are commonly caused by: excessive parallelism (MAXDOP too high), spinlock contention, or NUMA imbalance.
- Critical threshold: 7,500/sec per logical processor. Warning threshold: 5,000/sec per logical processor.
