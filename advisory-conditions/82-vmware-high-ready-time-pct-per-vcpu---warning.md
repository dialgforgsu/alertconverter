# VMware High Ready Time % per vCPU - Warning

## Overview
Warning-level variant of VMware High Ready Time % per vCPU. Alerts at a lower threshold (typically > 5% Ready Time per vCPU) to provide an early indication before reaching the critical threshold.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
Same mechanism as condition 81, with a lower threshold. SQL Sentry reads `cpu.ready` per vCPU via VMware integration and alerts at the warning level.

---

## Redgate Monitor Custom Metric

### Metric Name
Average Runnable Tasks Count (VMware Ready Time Proxy â Warning)

### Description
Reuse the metric from condition 81 (Average Runnable Tasks Count). Add a Medium severity threshold at a lower level to provide early warning of vCPU scheduling pressure before reaching the critical threshold.

> **Recommendation:** Redgate Monitor supports multiple severity thresholds on a single custom metric. Configure condition 81's metric with an additional Low/Medium threshold rather than creating a duplicate metric.

### Target Level
Instance

### T-SQL Query
```sql
-- Reuse the same query from condition 81
SELECT AVG(runnable_tasks_count) AS AvgRunnableTasksCount
FROM sys.dm_os_schedulers WITH (NOLOCK)
WHERE [status] = N'VISIBLE ONLINE';
```

### Configuration
Reuse the metric from condition 81. Add a Low/Medium severity threshold at the warning level.

### Alert Thresholds (Suggested)
| Severity | Threshold |
|---|---|
| High | > 10 (from condition 81 â critical) |
| Medium | > 5 (warning) |
| Low | > 2 |

### Notes
- See condition 81 (VMware High Ready Time % per vCPU) for full notes and remediation steps.
- Redgate Monitor supports High, Medium, and Low thresholds on a single custom metric â no need to duplicate metrics for warning vs critical variants.
- VMware Ready Time warning threshold: > 5% per vCPU. Critical threshold: > 10% per vCPU.
