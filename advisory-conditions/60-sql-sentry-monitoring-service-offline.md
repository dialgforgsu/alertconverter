# SQL Sentry Monitoring Service Offline

## Overview
Returns the SQL Sentry monitoring service that has been offline the longest in any site with actively watched connections. Only works when two or more monitoring services are used. Used to detect when the monitoring tool itself has failed.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT TOP 1 dbo.ManagementEngine.ServerName
FROM (SELECT Device.ID, Device.SiteID FROM dbo.Device WHERE IsPerformanceAnalysisEnabled = 1
    UNION SELECT Device.ID, Device.SiteID FROM dbo.EventSourceConnection
    INNER JOIN dbo.Device ON dbo.EventSourceConnection.DeviceID = dbo.Device.ID
    WHERE EventSourceConnection.IsWatched = 1 OR EventSourceConnection.IsPerformanceAnalysisEnabled = 1) WatchedDevices
INNER JOIN dbo.ManagementEngine ON WatchedDevices.SiteID = dbo.ManagementEngine.SiteID
WHERE dbo.ManagementEngine.HeartbeatDateTime < DATEADD(minute, -3, GETUTCDATE())
   OR dbo.ManagementEngine.HeartbeatDateTime IS NULL
ORDER BY ISNULL(dbo.ManagementEngine.HeartbeatDateTime, dbo.ManagementEngine.LastInitializationDateTime);
```

> **Note:** Uses SQL Sentry's internal monitoring database tables â not applicable as a standard SQL Server T-SQL query.

---

## Redgate Monitor Custom Metric

### Metric Name
Monitoring Heartbeat Check (Base Monitor Connectivity)

### Description
This condition cannot be replicated as a T-SQL custom metric because it detects failure of the monitoring tool itself. However, Redgate Monitor provides equivalent self-monitoring capability through its built-in alerting.

> **Caveat:** Redgate Monitor's Base Monitor service health is monitored through Redgate Monitor's own infrastructure alerting. If the Base Monitor stops collecting, the Web Server detects the collection gap and can alert accordingly. No custom metric is needed for this â configure the built-in "No data received" alert on critical servers.

### Target Level
Instance

### T-SQL Query
```sql
-- Connectivity check: returns 1 if this query executes successfully,
-- confirming the Base Monitor can reach this SQL Server instance.
-- Use as a simple liveness metric.
SELECT 1 AS BaseMonitorConnectivityCheck;
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 1 minute |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested)
Alert when this metric stops returning data â configure a "no data received" style alert in Redgate Monitor to detect collection gaps.

### Notes
- Redgate Monitor detects Base Monitor service outages through its own internal health monitoring.
- For multi-Base-Monitor estates, Redgate Monitor's Web Server shows the status of each Base Monitor.
- The built-in "Server went offline" alert in Redgate Monitor covers the scenario where a monitored SQL Server becomes unreachable.
