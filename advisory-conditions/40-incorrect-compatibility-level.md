# Incorrect Compatibility Level

## Overview
Evaluates to true when a database's compatibility level does not match the server's maximum supported level (derived from the master database). Databases running at lower compatibility levels miss query optimiser improvements available in the current SQL Server version.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
-- Uses SQL Sentry internal tables (PerformanceAnalysisSqlDatabase, EventSourceConnection)
SELECT CAST(pasd.CompatabilityLevel AS NVARCHAR(3)) + ' ' + esc.ServerName + ': (' + pasd.Name + ')',
    der.MaxCompatabilityLevel
FROM [dbo].[PerformanceAnalysisSqlDatabase] pasd
INNER JOIN [dbo].[EventSourceConnection] esc ON esc.ID = pasd.EventSourceConnectionID
INNER JOIN (SELECT EventSourceConnectionID, CompatabilityLevel AS MaxCompatabilityLevel
    FROM [dbo].[PerformanceAnalysisSqlDatabase] WHERE DatabaseID = 1) der
    ON der.EventSourceConnectionID = pasd.EventSourceConnectionID
WHERE esc.IsPerformanceAnalysisEnabled = 1
  AND pasd.CompatabilityLevel <> der.MaxCompatabilityLevel;
```

> **Note:** Uses SQL Sentry's internal monitoring database tables â not standard system views.

---

## Redgate Monitor Custom Metric

### Metric Name
Databases Below Max Compatibility Level

### Description
Returns the count of online user databases running below the server's maximum supported compatibility level. A non-zero count means those databases are missing query optimiser enhancements.

### Target Level
Instance

### T-SQL Query
```sql
DECLARE @maxCompatLevel INT;
SELECT @maxCompatLevel = compatibility_level
FROM sys.databases WHERE database_id = 1; -- master

SELECT COUNT(*) AS DatabasesBelowMaxCompatLevel
FROM sys.databases
WHERE database_id > 4
  AND state_desc = N'ONLINE'
  AND compatibility_level < @maxCompatLevel;
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 60 minutes |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested)
| Severity | Threshold |
|---|---|
| Medium | > 0 |

### Notes
- Some databases intentionally run at lower compat levels for application compatibility â exclude those or raise the threshold accordingly.
- Change compat level: `ALTER DATABASE [dbname] SET COMPATIBILITY_LEVEL = 160;` (2022=160, 2019=150, 2017=140, 2016=130).
- Always test workloads after a compat level change â query plans may change due to optimiser version differences.
