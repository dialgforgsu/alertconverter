# Suspect Pages - High Row Count

## Overview
The `msdb.dbo.suspect_pages` table is limited to 1,000 rows. When the count approaches 900, new suspect page events will not be logged, causing silent corruption to go undetected. This condition alerts before the table fills.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT count(*) AS suspectPageCount
FROM msdb.dbo.suspect_pages;
```

---

## Redgate Monitor Custom Metric

### Metric Name
Suspect Pages Table Row Count

### Description
Returns the total row count of the `msdb.dbo.suspect_pages` table. When this approaches 1,000, new corruption events will not be recorded. A high row count itself also indicates significant historical I/O issues.

### Target Level
Instance

### T-SQL Query
```sql
SELECT COUNT(*) AS SuspectPageRowCount
FROM msdb.dbo.suspect_pages;
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 15 minutes |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested)
| Severity | Threshold |
|---|---|
| High | > 900 |
| Medium | > 500 |
| Low | > 100 |

### Notes
- The `suspect_pages` table has a hard limit of 1,000 rows. Once full, SQL Server stops recording new suspect pages â corruption can go silently undetected.
- After resolving corruption, archive and clear old rows: delete rows for databases that have been restored or verified clean.
- A high row count is itself a red flag indicating persistent I/O problems on this server.
