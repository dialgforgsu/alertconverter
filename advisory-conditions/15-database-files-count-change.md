# Database Files Count Change

## Overview
Evaluates to true when the total number of database files changes. A count change may mean backup jobs need to be created or removed to cover new databases or filegroups.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT count(*) FROM sys.master_files;
```

---

## Redgate Monitor Custom Metric

### Metric Name
Total Database File Count

### Description
Returns the total count of all database files (data and log) across all databases. Enable rate-of-change alerting to detect file additions or removals.

### Target Level
Instance

### T-SQL Query
```sql
SELECT COUNT(*) AS TotalDatabaseFileCount
FROM sys.master_files;
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 30 minutes |
| Databases to collect from | master |
| Use calculated rate of change | Yes |

### Alert Thresholds (Suggested â rate of change)
| Severity | Threshold |
|---|---|
| Low | != 0 (any change) |

### Notes
- An increase may indicate a new database, new filegroup file, or added log file â review backup coverage.
- A decrease may indicate a dropped database or removed file â verify intentionality.
