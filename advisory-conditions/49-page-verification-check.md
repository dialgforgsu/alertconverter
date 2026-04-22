# Page Verification Check

## Overview
Evaluates to true when databases have an inadequate level of page protection. From SQL Server 2005 onwards, databases should use CHECKSUM page verification. Databases upgraded from older versions may still use TORN_PAGE_DETECTION.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT N'ALTER DATABASE [' + db.name + N'] SET PAGE_VERIFY CHECKSUM WITH NO_WAIT;'
FROM sys.databases AS db
WHERE db.page_verify_option_desc <> N'CHECKSUM';
```

---

## Redgate Monitor Custom Metric

### Metric Name
Databases Without CHECKSUM Page Verification

### Description
Returns the count of online user databases not using CHECKSUM page verification. Databases using TORN_PAGE_DETECTION or NONE offer weaker I/O corruption detection.

### Target Level
Instance

### T-SQL Query
```sql
SELECT COUNT(*) AS DatabasesWithoutChecksum
FROM sys.databases
WHERE database_id > 4
  AND state_desc = N'ONLINE'
  AND page_verify_option_desc <> N'CHECKSUM';
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
| High | > 0 |

### Notes
- Fix with: `ALTER DATABASE [dbname] SET PAGE_VERIFY CHECKSUM WITH NO_WAIT;`
- To generate fix scripts for all affected databases:
  ```sql
  SELECT N'ALTER DATABASE [' + name + N'] SET PAGE_VERIFY CHECKSUM WITH NO_WAIT;'
  FROM sys.databases WHERE page_verify_option_desc <> N'CHECKSUM' AND database_id > 4;
  ```
- CHECKSUM detects far more corruption scenarios than TORN_PAGE_DETECTION and has minimal overhead.
