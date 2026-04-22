# Sleeping Sessions With Old Open Transactions

## Overview
Detects sleeping sessions with open transactions older than 10 minutes. Such sessions can cause blocking, prevent transaction log truncation (leading to log growth), and block TempDB version store cleanup when snapshot isolation is used.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT TxDesc = '[' + CONVERT(varchar, es.last_request_start_time, 120) + '] (' + CAST(es.session_id AS varchar(6)) + ') '
    + host_name + ':' + program_name + ' [' + DB_NAME(dt.database_id) + ']',
    OpenMinutes = DATEDIFF(minute, es.last_request_start_time, GETDATE())
FROM sys.dm_exec_sessions es
JOIN sys.dm_tran_session_transactions st ON es.session_id = st.session_id
JOIN sys.dm_tran_database_transactions dt ON dt.transaction_id = st.transaction_id
WHERE dt.database_id <> 32767
  AND status = 'sleeping'
  AND es.last_request_start_time < DATEADD(MINUTE, -5, GETDATE())
ORDER BY es.last_request_start_time;
```

---

## Redgate Monitor Custom Metric

### Metric Name
Sleeping Sessions With Old Open Transactions (Minutes)

### Description
Returns the age in minutes of the oldest sleeping session with an open transaction. Sleeping sessions with long-held transactions are a common cause of log growth, blocking, and TempDB version store bloat.

### Target Level
Instance

### T-SQL Query
```sql
SELECT ISNULL(
    (SELECT MAX(DATEDIFF(MINUTE, es.last_request_start_time, GETDATE()))
     FROM sys.dm_exec_sessions AS es
     INNER JOIN sys.dm_tran_session_transactions AS st
         ON es.session_id = st.session_id
     INNER JOIN sys.dm_tran_database_transactions AS dt
         ON dt.transaction_id = st.transaction_id
     WHERE dt.database_id <> 32767
       AND es.status = N'sleeping'
       AND es.last_request_start_time < DATEADD(MINUTE, -5, GETDATE())), 0)
        AS OldestSleepingTxMinutes;
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 2 minutes |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested)
| Severity | Threshold |
|---|---|
| High | > 60 minutes |
| Medium | > 20 minutes |
| Low | > 10 minutes |

### Notes
- The most common cause is applications that open transactions, perform work, and then forget to commit or rollback before the connection goes idle.
- Kill the offending session with `KILL <session_id>;` after confirming with the application team.
- Connection pooling frameworks should be configured with transaction timeout settings to prevent this pattern.
