# VMware High Co-Stop %

## Overview
Co-Stop time is the time a virtual machine is ready to run but cannot because of co-scheduling constraints (multiple vCPUs must be scheduled simultaneously). Values above 3% indicate too many vCPUs are assigned relative to available physical CPUs.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
No T-SQL â SQL Sentry reads VMware `cpu.costop` counter via its VMware integration. Alerts when Co-Stop % > 3%.

---

## Redgate Monitor Custom Metric

### Metric Name
Average Runnable Tasks Count (VMware Co-Stop Proxy)

### Description
VMware co-stop counters are not accessible via T-SQL. This metric uses SQL Server's runnable tasks count as a proxy â co-stop prevents vCPU execution, which causes SQL Server threads to accumulate in the runnable queue.

> **Caveat:** For genuine VMware co-stop monitoring, use `esxtop` or VMware performance counters (`cpu.costop`). This metric surfaces the SQL Server impact of co-stop pressure.

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
- Co-stop primarily affects VMs with high vCPU counts (>= 4 vCPUs). Reduce vCPU count to the actual parallelism requirement.
- A SQL Server VM rarely needs more than 4-8 vCPUs â MAXDOP should be set to limit parallelism accordingly.
- For definitive co-stop measurement use VMware's `esxtop` tool or vSphere performance charts.
