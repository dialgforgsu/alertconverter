# High Context Switches - Warning

## Overview
Warning-level variant of High Context Switches. Values over 5,000 per logical processor are an early indicator of excessive thread switching overhead.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
Same mechanism as condition 23, with a lower threshold of 5,000 context switches per second per logical processor.

---

## Redgate Monitor Custom Metric

### Metric Name
Average Runnable Tasks Count (Warning Level)

### Description
Reuse the metric from condition 23 (Average Runnable Tasks Count). Add a Medium severity alert threshold at the lower warning level. No separate metric definition is needed.

> **Recommendation:** Redgate Monitor supports multiple severity thresholds (High, Medium, Low) on a single custom metric. Configure condition 23's metric with an additional Medium threshold rather than creating a duplicate metric.

### Target Level
Instance

### T-SQL Query
```sql
-- Reuse the same query from condition 23
SELECT AVG(runnable_tasks_count) AS AvgRunnableTasksCount
FROM sys.dm_os_schedulers WITH (NOLOCK)
WHERE [status] = N'VISIBLE ONLINE';
```

### Configuration
Reuse the metric from condition 23. Add a Medium severity threshold at the warning level.

### Alert Thresholds (Suggested)
| Severity | Threshold |
|---|---|
| High | > 10 (from condition 23) |
| Medium | > 3 |
| Low | > 1 |

### Notes
- See condition 23 (High Context Switches) for full notes and remediation steps.
- Redgate Monitor supports High, Medium, and Low thresholds on a single custom metric â use all three tiers on one metric rather than duplicating.
