# VMware High Ready Time % per vCPU

## Overview
Ready Time accumulates when a vCPU is ready to do work but is waiting for the hypervisor to schedule it on a physical CPU. This is often caused by overprovisioning vCPUs relative to physical CPUs on the host or VMs of significantly different sizes competing for the same physical cores.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
No T-SQL â SQL Sentry reads VMware `cpu.ready` counter per vCPU via its VMware integration. Alerts at the critical threshold (typically > 10% Ready Time per vCPU).

---

## Redgate Monitor Custom Metric

### Metric Name
Average Runnable Tasks Count (VMware Ready Time Proxy â Critical)

### Description
VMware CPU Ready Time per vCPU is not accessible via T-SQL. This metric uses SQL Server's runnable tasks count as a proxy â when vCPUs are waiting for the hypervisor to schedule them, SQL Server's runnable queue grows as threads are unable to execute.

> **Caveat:** For genuine VMware Ready Time monitoring, use VMware vSphere performance counters (`cpu.ready`) or `esxtop`. The threshold for critical Ready Time is typically > 10% per vCPU. This metric surfaces the SQL Server impact.

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
| Medium | > 5 |

### Notes
- VMware CPU Ready Time > 5% per vCPU is considered a warning; > 10% is critical.
- Resolution: reduce vCPU count on the VM, reduce the vCPU-to-pCPU overcommit ratio on the host, or migrate the VM to a less-loaded host.
- A SQL Server VM rarely benefits from more than 4-8 vCPUs â excess vCPUs increase co-scheduling pressure without improving performance.
- Check Redgate Monitor's Activity Graph for any available VMware guest CPU counters surfaced via WMI.
