# Hyper-V High vCPU Wait Time

## Overview
vCPU Wait Time is a SQL Sentry virtual counter: CPU wait time per dispatch (ns) multiplied by dispatches per second. High values indicate the Hyper-V host is overprovisioned or under-resourced, causing vCPU scheduling delays.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
No T-SQL â SQL Sentry reads Hyper-V guest CPU counters via Windows PerfMon: `Hyper-V Hypervisor Virtual Processor: CPU Wait Time Per Dispatch` and `Hyper-V Hypervisor Virtual Processor: Hypercalls/sec`.

---

## Redgate Monitor Custom Metric

### Metric Name
Average Runnable Tasks Count (Hyper-V CPU Pressure Proxy)

### Description
Hyper-V vCPU wait time is not accessible via T-SQL. This metric uses SQL Server's runnable tasks count as a proxy â when the Hyper-V host is CPU-starved, SQL Server's runnable queue grows because the hypervisor cannot schedule vCPUs promptly.

> **Caveat:** Redgate Monitor's WMI collection may surface Hyper-V guest CPU counters in the Activity Graph if they are registered on the guest OS. Check the Activity Graph â Hyper-V category if available.

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
- For genuine Hyper-V vCPU wait time, collect `Hyper-V Hypervisor Virtual Processor` PerfMon counters from the host or guest OS.
- Common resolutions: reduce vCPU overcommit ratio, pin SQL Server VM to specific NUMA nodes, migrate VM to a less-loaded host.
- Reuses the same query as condition 37 (High Runnable Tasks Count) â no separate metric required if that condition is already deployed.
