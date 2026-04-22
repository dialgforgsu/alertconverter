# Service Broker - Login Connection Error

## Overview
Checks `sys.dm_broker_connections` for a `login_state` of 13, which indicates a connection error. Login errors prevent Service Broker from establishing transport connections between SQL Server instances.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
SQL Sentry queries `sys.dm_broker_connections` for `login_state = 13`.

---

## Redgate Monitor Custom Metric

### Metric Name
Service Broker Connection Errors

### Description
Returns the count of Service Broker connections in a login error state (`login_state = 13`). These connections cannot be established, preventing cross-instance message delivery.

### Target Level
Instance

### T-SQL Query
```sql
SELECT COUNT(*) AS BrokerConnectionErrors
FROM sys.dm_broker_connections
WHERE login_state = 13;
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
- Login state 13 indicates a transport-level login error â typically a certificate trust issue, Windows authentication problem, or Service Broker endpoint misconfiguration.
- Check the SQL Server error log for detailed Service Broker connection error messages.
- Verify Service Broker endpoint certificates are valid and trusted on both endpoints.
- Use `SELECT * FROM sys.dm_broker_connections;` to inspect all connection states and peer addresses.
