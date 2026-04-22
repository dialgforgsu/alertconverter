# High Recompiles

## Overview
Query plan recompiles should be < 15% of initial compiles and often correlate with high CPU. Caused by statistics updates, schema changes, SET option changes, or temporary table modifications.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
No standalone T-SQL â SQL Sentry reads `SQLServer:SQL Statistics - SQL Re-Compilations/sec` via Windows PerfMon.

---

## Redgate Monitor Custom Metric

### Metric Name
SQL Re-Compilations per Second

### Description
Returns the current SQL Re-Compilations/sec counter. Persistently high values indicate queries are being frequently recompiled, consuming unnecessary CPU and destabilising execution plans.

### Target Level
Instance

### T-SQL Query
```sql
SELECT cntr_value AS SqlReCompilationsPerSec
FROM sys.dm_os_performance_counters
WHERE [object_name] LIKE N'%SQL Statistics%'
  AND counter_name = N'SQL Re-Compilations/sec';
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
| High | > 500 |
| Medium | > 200 |
| Low | > 100 |

### Notes
- Common causes: statistics auto-updates, DDL schema changes, SET option mismatches, table variable cardinality, temp table plan invalidation.
- Use Redgate Monitor's Top Queries view sorted by recompile count to find the most frequently recompiling statements.
- Excessive recompiles often indicate a need for `OPTION (KEEPFIXED PLAN)` or improved statistics management.
