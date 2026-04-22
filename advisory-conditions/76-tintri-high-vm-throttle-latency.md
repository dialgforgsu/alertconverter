# Tintri High VM Throttle Latency

## Overview
Checks for Tintri Virtual Machine throttle latency > 10 ms on any VM. VM-level throttling means the entire virtual machine's storage I/O is being rate-limited by the Tintri array.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
No T-SQL â SQL Sentry reads `Tintri Virtual Machine: Throttle Latency (ms)` via its Tintri integration.

---

## Redgate Monitor Custom Metric

### Metric Name
Combined I/O Stall Rate (ms) â Tintri VM Throttle Proxy

### Description
Tintri VM-level throttle counters are not accessible via T-SQL. This metric returns the overall I/O stall across all database files (per collection delta) as the most comprehensive proxy for VM-level storage throttling impact on SQL Server.

> **Caveat:** For genuine Tintri VM throttle monitoring, use Tintri's management API or native monitoring tools.

### Target Level
Instance

### T-SQL Query
```sql
SELECT SUM(fs.io_stall) AS TotalIOStallMs
FROM sys.dm_io_virtual_file_stats(NULL, NULL) AS fs
INNER JOIN sys.master_files AS mf
    ON fs.database_id = mf.database_id AND fs.[file_id] = mf.[file_id]
WHERE mf.state_desc = N'ONLINE';
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 2 minutes |
| Databases to collect from | master |
| Use calculated rate of change | Yes |

### Alert Thresholds (Suggested â rate of change, ms per collection interval)
| Severity | Threshold |
|---|---|
| High | > 120000 ms/interval |
| Medium | > 30000 ms/interval |

### Notes
- `io_stall` is cumulative â use rate-of-change to detect sudden I/O stall increases.
- Total I/O stall growth correlates with VM-level storage throttling as all files on the throttled VM will be affected simultaneously.
- Redgate Monitor's Activity Graph shows the OS-level disk queue length and latency counters collected via WMI â use these for storage-layer diagnosis.
