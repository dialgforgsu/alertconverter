# High Compiles + High CPU

## Overview
Compound condition â fires when both CPU utilisation and SQL compilations/sec are simultaneously elevated. Plan compilations are CPU-intensive and may be causing or contributing to high CPU.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
No single T-SQL query â SQL Sentry combines `SQLServer:SQL Statistics - SQL Compilations/sec` with CPU % (both via Windows PerfMon/WMI). Both sub-conditions must be true simultaneously.

---

## Redgate Monitor Custom Metric

### Metric Name
SQL Compilations per Second (for CPU correlation)

### Description
Returns SQL Compilations/sec. Use Redgate Monitor's Analysis graph to overlay this against the built-in CPU utilisation counter to visually confirm the correlation.

> **Note:** Redgate Monitor's built-in CPU alert covers the CPU side. This custom metric provides the compilation signal. Correlate the two using the Analysis graph.

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
| High | > 3000 |
| Medium | > 1500 |

### Notes
- This is the same query as condition 21. Reuse that metric definition rather than creating a duplicate.
- If both metrics fire together, investigate: parameter sniffing, schema change recompiles, statistics update recompiles, insufficient plan cache memory.
