# Suspect Pages - Increase in Fixed Pages

## Overview
Queries `msdb.dbo.suspect_pages` for fixed pages (event types 4, 5, 7 â DBCC repaired, automatic page repair, deallocated). An increase means DBCC CHECKDB corrected pages or automatic page repair occurred, indicating potential underlying hardware issues.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT db_name(database_id) AS databaseName, count(*) AS fixedPages
FROM msdb.dbo.suspect_pages
WHERE event_type IN (4, 5, 7)
GROUP BY database_id;
```

---

## Redgate Monitor Custom Metric

### Metric Name
Fixed Suspect Pages Count

### Description
Returns the count of suspect pages that have been fixed (event types 4, 5, or 7). An increase in this metric means pages were corrected â either by DBCC CHECKDB repair or automatic page repair â which signals underlying hardware or storage issues that may cause new corruption.

### Target Level
Instance

### T-SQL Query
```sql
SELECT COUNT(*) AS FixedSuspectPages
FROM msdb.dbo.suspect_pages
WHERE event_type IN (4, 5, 7);
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 30 minutes |
| Databases to collect from | master |
| Use calculated rate of change | Yes â alert on increase |

### Alert Thresholds (Suggested â Rate of Change)
| Severity | Threshold |
|---|---|
| Medium | > 0 (any increase) |

### Notes
- Event types: 4 = error resolved via DBCC CHECKDB repair, 5 = automatic page repair (Enterprise AG/Mirroring), 7 = page deallocated by DBCC.
- Fixed pages indicate the immediate corruption was resolved but the root cause (bad disk, SAN issue, memory error) likely persists.
- Investigate the underlying hardware after any fixed page event â run extended memory diagnostics and disk health checks.
