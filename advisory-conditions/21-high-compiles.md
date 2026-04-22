# High Compiles

## Overview
Query plan compiles should be < 15% of batch requests/sec. Higher values indicate low plan reuse, often correlating with high CPU since compilation is CPU-intensive. May indicate memory pressure preventing plan retention.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
No standalone T-SQL â SQL Sentry reads `SQLServer:SQL Statistics - SQL Compilations/sec` via Windows PerfMon.

---

## Redgate Monitor Custom Metric

### Metric Name
SQL Compilations per Second

### Description
Returns the current SQL Compilations/sec counter. Pair with the Batch Requests/sec metric to monitor the compile-to-batch ratio.

### Target Level
Instance

### T-SQL Query
```sql
SELECT cntr_value AS SqlCompilationsPerSec
FROM sys.dm_os_performance_counters
WHERE [object_name] LIKE N'%SQL Statistics%'
  AND counter_name = N'SQL Compilations/sec';
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
| High | > 5000 |
| Medium | > 2000 |
| Low | > 1000 |

### Notes
- `cntr_value` for `/sec` counters is already a running average â no rate-of-change calculation needed.
- High compiles + high CPU strongly suggests plan cache issues â check single-use plan cache (condition 18) and consider enabling `optimize for ad hoc workloads`.
- Create a companion metric for `Batch Requests/sec` and compare the ratio using Redgate Monitor's Analysis graph.
