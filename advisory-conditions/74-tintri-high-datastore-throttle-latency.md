# Tintri High Datastore Throttle Latency

## Overview
Checks for Tintri Datastore throttle latency > 10 ms. Throttle latency on a Tintri storage system means the storage array is actively rate-limiting I/O to protect QoS, indicating storage overload.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
No T-SQL â SQL Sentry reads Tintri-specific performance counters via its Tintri integration. The counter is `Tintri Datastore: Throttle Latency (ms)`.

---

## Redgate Monitor Custom Metric

### Metric Name
Max Data File Read Latency (ms) â Tintri Storage Proxy

### Description
Tintri-specific storage counters are not accessible via T-SQL. This metric uses SQL Server's measured I/O stall latency as a proxy â if Tintri is throttling, SQL Server will observe elevated read/write latency on affected files.

> **Caveat:** This is not a Tintri-specific metric. For genuine Tintri datastore throttle monitoring, use Tintri's management API or a monitoring tool with Tintri integration. This metric surfaces the SQL Server impact of storage throttling.

### Target Level
Instance

### T-SQL Query
```sql
SELECT MAX(
    CASE WHEN fs.num_of_reads + fs.num_of_writes = 0 THEN 0
    ELSE CAST(fs.io_stall AS DECIMAL(18,2)) / (fs.num_of_reads + fs.num_of_writes)
    END) AS MaxAvgIOLatencyMs
FROM sys.dm_io_virtual_file_stats(NULL, NULL) AS fs
INNER JOIN sys.master_files AS mf
    ON fs.database_id = mf.database_id AND fs.[file_id] = mf.[file_id];
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
| High | > 50 ms |
| Medium | > 20 ms |
| Low | > 10 ms |

### Notes
- For genuine Tintri Datastore throttle latency monitoring, use Tintri's VMstore REST API or Tintri's native monitoring tools.
- SQL Server I/O latency is the downstream impact of Tintri throttling â high latency here in conjunction with Tintri throttle events confirms storage is the bottleneck.
