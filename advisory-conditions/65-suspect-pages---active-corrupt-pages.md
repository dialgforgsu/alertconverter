# Suspect Pages - Active Corrupt Pages

## Overview
Queries `msdb.dbo.suspect_pages` for pages with event_type 1, 2, or 3 (823/824 errors, bad checksum, torn page). Any active corrupt pages indicate potential database corruption requiring immediate attention.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT db_name(database_id) AS databaseName, count(*) AS corruptPages
FROM msdb.dbo.suspect_pages
WHERE event_type IN (1,2,3)
GROUP BY database_id;
```

---

## Redgate Monitor Custom Metric

### Metric Name
Active Corrupt Suspect Pages

### Description
Returns the total count of pages with active corruption indicators (event_type 1, 2, or 3) across all databases. Any non-zero value indicates possible database corruption requiring immediate investigation with DBCC CHECKDB.

### Target Level
Instance

### T-SQL Query
```sql
SELECT COUNT(*) AS ActiveCorruptPages
FROM msdb.dbo.suspect_pages
WHERE event_type IN (1, 2, 3);
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 5 minutes |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested)
| Severity | Threshold |
|---|---|
| High | > 0 |

### Notes
- Event types: 1 = 823 error (hard I/O error), 2 = 824 error (logical consistency error), 3 = torn page.
- Immediately run `DBCC CHECKDB([database_name]) WITH NO_INFOMSGS;` on affected databases.
- If corruption is confirmed, restore from a known-good backup. Do not attempt repair without a current backup.
- On Enterprise Edition with AG or Mirroring, automatic page repair may resolve the issue â check for event_type 5 (page repaired) entries.
