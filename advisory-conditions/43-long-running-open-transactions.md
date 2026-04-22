# Long Running Open Transactions

## Overview
Evaluates to true if any running transaction has been open for at least 90 seconds. Long-running open transactions can cause blocking, prevent log truncation, and block version store cleanup in TempDB when snapshot isolation is used.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT [Last T-SQL Text], [Transaction Length]
FROM (
    SELECT s_tst.session_id, s_es.login_name,
        DB_NAME(s_tdt.database_id) AS [Database],
        s_tdt.database_transaction_begin_time AS [Begin Time],
        DATEDIFF(ss, s_tdt.database_transaction_begin_time, GETDATE()) AS [Transaction Length],
        s_tdt.database_transaction_log_bytes_used AS [Log Bytes],
        s_est.text AS [Last T-SQL Text]
    FROM sys.dm_tran_database_transactions s_tdt
    INNER JOIN sys.dm_tran_session_transactions s_tst ON s_tst.transaction_id = s_tdt.transaction_id
    INNER JOIN sys.dm_exec_sessions s_es ON s_es.session_id = s_tst.session_id
    INNER JOIN sys.dm_exec_connections s_ec ON s_ec.session_id = s_tst.session_id
    CROSS APPLY sys.dm_exec_sql_text(s_ec.most_recent_sql_handle) AS s_est
    WHERE s_tst.is_user_transaction = 1
) RunningTransactions
WHERE [Transaction Length] IS NOT NULL
ORDER BY [Transaction Length] DESC;
```

---

## Redgate Monitor Custom Metric

### Metric Name
Longest Open Transaction (Seconds)

### Description
Returns the age in seconds of the oldest currently open user transaction. A value above 90 seconds warrants investigation for blocking or log growth impact.

### Target Level
Instance

### T-SQL Query
```sql
SELECT ISNULL(
    (SELECT MAX(DATEDIFF(SECOND, s_tdt.database_transaction_begin_time, GETDATE()))
     FROM sys.dm_tran_database_transactions s_tdt
     INNER JOIN sys.dm_tran_session_transactions s_tst
         ON s_tst.transaction_id = s_tdt.transaction_id
     INNER JOIN sys.dm_exec_sessions s_es
         ON s_es.session_id = s_tst.session_id
     WHERE s_tst.is_user_transaction = 1
       AND s_tdt.database_id <> 32767), 0) AS LongestOpenTransactionSeconds;
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 1 minute |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested)
| Severity | Threshold |
|---|---|
| High | > 600 seconds (10 min) |
| Medium | > 300 seconds (5 min) |
| Low | > 90 seconds |

### Notes
- Long-running transactions prevent log truncation, causing log file growth.
- When snapshot isolation is active, long transactions prevent version store cleanup, causing TempDB growth.
- Use Redgate Monitor's blocking chain visualisation to identify downstream victims of long-running transactions.
