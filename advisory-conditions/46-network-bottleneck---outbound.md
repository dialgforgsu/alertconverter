# Network Bottleneck - Outbound

## Overview
Output Queue Length is the size of the network output packet queue. A sustained value of more than 3 packets may indicate a network bottleneck. This condition detects a queue length > 3 on any network adapter.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
No T-SQL â SQL Sentry reads `Network Interface: Output Queue Length` via Windows WMI/PerfMon. Alerts when any adapter's queue length > 3.

---

## Redgate Monitor Custom Metric

### Metric Name
Network Send Wait Count (SQL Server Proxy)

### Description
Windows network output queue length is not accessible via T-SQL. This metric uses SQL Server's network-related wait statistics as a proxy for outbound network pressure.

> **Caveat:** Redgate Monitor's WMI collection surfaces `Network Interface: Output Queue Length` in the Activity Graph natively. Use that counter for definitive network queue monitoring; this metric provides a supplementary SQL-level signal.

### Target Level
Instance

### T-SQL Query
```sql
SELECT SUM(wait_time_ms) AS NetworkWaitTimeMs
FROM sys.dm_os_wait_stats
WHERE wait_type IN (
    N'ASYNC_NETWORK_IO',
    N'NET_WAITFOR_PACKET',
    N'NETWORK_IO'
);
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 1 minute |
| Databases to collect from | master |
| Use calculated rate of change | Yes |

### Alert Thresholds (Suggested â rate of change, ms/minute)
| Severity | Threshold |
|---|---|
| High | > 60000 ms/min |
| Medium | > 10000 ms/min |

### Notes
- `ASYNC_NETWORK_IO` is the most common indicator of clients not consuming results fast enough (often a slow client or saturated network).
- Check Redgate Monitor's Activity Graph â Network category for the OS-level `Output Queue Length` counter.
- High `ASYNC_NETWORK_IO` combined with a high output queue length confirms a genuine network bottleneck.
