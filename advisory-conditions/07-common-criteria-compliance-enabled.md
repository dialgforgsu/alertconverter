# Common Criteria Compliance Enabled

## Overview
Queries `sys.configurations` to determine if Common Criteria Compliance is enabled. This advanced option (Enterprise/Datacenter only) can dramatically impact server performance by enabling residue information protection and enhanced login auditing. Activation requires a server restart.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT value
FROM sys.configurations
WHERE name = N'common criteria compliance enabled'
```

---

## Redgate Monitor Custom Metric

### Metric Name
Common Criteria Compliance Enabled

### Description
Returns 1 if Common Criteria Compliance is currently active (`value_in_use = 1`), 0 otherwise. This setting adds significant overhead and should only be enabled when required for a compliance mandate.

### Target Level
Instance

### T-SQL Query
```sql
SELECT CAST(value_in_use AS INT) AS CommonCriteriaEnabled
FROM sys.configurations
WHERE name = N'common criteria compliance enabled';
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
| High | > 0 (i.e., = 1, enabled) |

### Notes
- `value_in_use` reflects the active state; `value` reflects the pending configured state (requires restart to take effect).
- Disable with: `EXEC sp_configure 'common criteria compliance enabled', 0; RECONFIGURE;` then restart SQL Server.
- Effects include: memory page zeroing on deallocation, enhanced login auditing, last successful login tracking â all add CPU/memory overhead.
