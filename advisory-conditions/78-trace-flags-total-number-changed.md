# Trace Flags Total Number Changed

## Overview
Evaluates to true when the total number of trace flags visible to `DBCC TRACESTATUS` has changed â whether enabled or disabled. Tracks any addition or removal of trace flags from the system.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
CREATE TABLE [#TraceFlags] ([TraceFlag] INT, [Status] INT, [Global] INT, [Session] INT);
INSERT INTO #TraceFlags EXEC ('DBCC TRACESTATUS WITH NO_INFOMSGS');
SELECT Count(*) AS TraceFlagsTotal FROM #TraceFlags;
DROP TABLE [#TraceFlags];
```

---

## Redgate Monitor Custom Metric

### Metric Name
Total Visible Trace Flag Count

### Description
Returns the total count of all trace flags visible via `DBCC TRACESTATUS` (both enabled and disabled). Use rate-of-change alerting to detect when trace flags are added to or removed from the server's configuration.

### Target Level
Instance

### T-SQL Query
```sql
CREATE TABLE #tf2 (TraceFlag INT, [Status] INT, [Global] INT, [Session] INT);
INSERT INTO #tf2 EXEC ('DBCC TRACESTATUS WITH NO_INFOMSGS');
SELECT COUNT(*) AS TotalTraceFlagCount FROM #tf2;
DROP TABLE #tf2;
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
- This metric is broader than condition 77 (Enabled Trace Flag Count) â it catches trace flags being added or removed even if their status doesn't change.
- Session-level trace flags (set without `-1`) only appear in `DBCC TRACESTATUS` output for the current session â this metric counts global trace flags only.
- Startup trace flags (set via `-T` in the SQL Server service startup parameters) will always appear enabled here.
