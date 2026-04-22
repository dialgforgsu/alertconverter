# Trace Flags Number Turned On Changed

## Overview
Evaluates to true when the number of trace flags set to ON (status = 1) has changed since the last check. Unexpected trace flag activation may indicate an unauthorised configuration change or diagnostic activity by another party.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
CREATE TABLE [#TraceFlagsOn] ([TraceFlag] INT, [Status] INT, [Global] INT, [Session] INT);
INSERT INTO #TraceFlagsOn EXEC ('DBCC TRACESTATUS WITH NO_INFOMSGS');
SELECT Count(*) AS TraceFlagsStatusOn FROM #TraceFlagsOn WHERE [#TraceFlagsOn].Status = 1;
DROP TABLE [#TraceFlagsOn];
```

---

## Redgate Monitor Custom Metric

### Metric Name
Enabled Trace Flag Count

### Description
Returns the count of trace flags currently set to ON (Status = 1). Use rate-of-change alerting to detect when trace flags are enabled or disabled unexpectedly.

### Target Level
Instance

### T-SQL Query
```sql
CREATE TABLE #tf (TraceFlag INT, [Status] INT, [Global] INT, [Session] INT);
INSERT INTO #tf EXEC ('DBCC TRACESTATUS WITH NO_INFOMSGS');
SELECT COUNT(*) AS EnabledTraceFlagCount FROM #tf WHERE [Status] = 1;
DROP TABLE #tf;
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 30 minutes |
| Databases to collect from | master |
| Use calculated rate of change | Yes â alert on any change |

### Alert Thresholds (Suggested â Rate of Change)
| Severity | Threshold |
|---|---|
| Low | != 0 (any change) |

### Notes
- Trace flags can be enabled/disabled with `DBCC TRACEON(<flag>, -1);` and `DBCC TRACEOFF(<flag>, -1);`
- An unexpected increase may mean someone is running diagnostics. An unexpected decrease may mean a startup flag was removed.
- Common production trace flags (SQL Server 2016 and earlier): TF 1117, TF 1118 (TempDB), TF 2371 (dynamic statistics), TF 4199 (query processor hotfixes).
