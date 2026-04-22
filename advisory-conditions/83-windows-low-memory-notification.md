# Windows Low Memory Notification

## Overview
Queries the SQL Server ring buffer for recent occurrences of RESOURCE_MEMPHYSICAL_LOW alerts from the Windows Memory Manager. These occur when Windows signals that physical memory is critically low, causing SQL Server to release buffer pool memory in response.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT COUNT(*) AS AlertCount
FROM (
    SELECT CAST(orb.record AS XML) AS xmlRec
    FROM sys.dm_os_ring_buffers AS orb
    CROSS JOIN sys.dm_os_sys_info AS osi
    WHERE orb.ring_buffer_type = N'RING_BUFFER_RESOURCE_MONITOR'
    AND DATEADD(second, -((osi.cpu_ticks/(osi.cpu_ticks/osi.ms_ticks) - orb.timestamp) / 1000), GETDATE())
        > DATEADD(minute, -6, GETDATE())
) rb
CROSS APPLY rb.xmlRec.nodes('Record') rec(x)
WHERE rec.x.value('(ResourceMonitor/IndicatorsSystem)[1]','tinyint') = 2
```

---

## Redgate Monitor Custom Metric

### Metric Name
Windows Low Memory Notifications (Last 6 Minutes)

### Description
Returns the count of RESOURCE_MEMPHYSICAL_LOW notifications in the SQL Server ring buffer within the last 6 minutes. Each notification means Windows told SQL Server that physical memory was critically low. Any non-zero count indicates active severe memory pressure.

### Target Level
Instance

### T-SQL Query
```sql
SELECT COUNT(*) AS LowMemoryNotificationCount
FROM (
    SELECT CAST(orb.record AS XML) AS xmlRec
    FROM sys.dm_os_ring_buffers AS orb
    CROSS JOIN sys.dm_os_sys_info AS osi
    WHERE orb.ring_buffer_type = N'RING_BUFFER_RESOURCE_MONITOR'
      AND DATEADD(
            SECOND,
            -CAST(((osi.cpu_ticks / (osi.cpu_ticks / osi.ms_ticks)) - orb.timestamp) / 1000 AS INT),
            GETDATE()
          ) > DATEADD(MINUTE, -6, GETDATE())
) AS rb
CROSS APPLY rb.xmlRec.nodes('Record') AS rec(x)
WHERE rec.x.value('(ResourceMonitor/IndicatorsSystem)[1]', 'TINYINT') = 2;
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
| High | > 3 |
| Medium | > 0 |

### Notes
- `IndicatorsSystem = 2` corresponds to RESOURCE_MEMPHYSICAL_LOW â the Windows physical memory low signal.
- When this fires, SQL Server shrinks the buffer pool, causing Page Life Expectancy to drop and disk I/O to increase as pages are evicted.
- Pair with condition 44 (Low Available Windows Memory) and condition 45 (Low Page Life Expectancy) for a complete memory pressure picture.
- Resolution: increase `max server memory` headroom for the OS, add physical RAM, reduce non-SQL Server memory consumers, or investigate memory leaks.
