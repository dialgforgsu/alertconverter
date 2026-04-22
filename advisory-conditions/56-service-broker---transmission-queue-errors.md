# Service Broker - Transmission Queue Errors

## Overview
Counts the number of messages in `sys.transmission_queue` where `is_conversation_error = 1`. Messages stuck in the transmission queue with errors indicate delivery failures.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
SQL Sentry counts rows in `sys.transmission_queue` where `is_conversation_error = 1`.

---

## Redgate Monitor Custom Metric

### Metric Name
Service Broker Transmission Queue Error Count

### Description
Returns the count of messages in the Service Broker transmission queue that have conversation errors. These messages are stuck and cannot be delivered to their destination.

### Target Level
Instance

### T-SQL Query
```sql
SELECT COUNT(*) AS TransmissionQueueErrors
FROM sys.transmission_queue
WHERE is_conversation_error = 1;
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
| High | > 10 |
| Medium | > 0 |

### Notes
- `transmission_status` column contains the error description for stuck messages â query it directly to diagnose the failure.
- Common causes: network connectivity issues, certificate problems, endpoint misconfiguration, or destination queue being disabled.
- Messages with errors can be queried with: `SELECT *, transmission_status FROM sys.transmission_queue WHERE is_conversation_error = 1;`
