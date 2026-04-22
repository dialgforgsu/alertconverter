# VMware High Ballooning

## Overview
VMware memory ballooning occurs when the hypervisor reclaims memory from guest VMs. Significant ballooning means Windows and SQL Server are losing physical memory to the hypervisor, causing performance degradation equivalent to physical memory shortage.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
No T-SQL â SQL Sentry reads VMware balloon driver memory counters via its VMware integration. The counter reflects memory reclaimed by the VMware balloon driver.

---

## Redgate Monitor Custom Metric

### Metric Name
SQL Server Physical Memory Low Flag (VMware Ballooning Proxy)

### Description
VMware balloon driver metrics are not accessible via T-SQL. This metric monitors SQL Server's response to physical memory pressure as a proxy â when VMware is ballooning aggressively, SQL Server will observe low physical memory and shrink the buffer pool.

> **Caveat:** For genuine VMware ballooning monitoring, use VMware vSphere performance counters (`mem.vmmemctl`) from the hypervisor or guest OS. This metric shows the SQL Server impact.

### Target Level
Instance

### T-SQL Query
```sql
SELECT CAST(process_physical_memory_low AS INT) AS PhysicalMemoryLow,
       CAST(available_physical_memory_kb AS DECIMAL(18,2)) / 1024.0 AS AvailableMemoryMB
FROM sys.dm_os_process_memory
CROSS JOIN sys.dm_os_sys_memory;
```

### Configuration
| Setting | Value |
|---|---|
| Collection frequency | 1 minute |
| Databases to collect from | master |
| Use calculated rate of change | No |

### Alert Thresholds (Suggested)
Alert when `PhysicalMemoryLow` = 1 OR `AvailableMemoryMB` < 500.

### Notes
- VMware balloon driver (`vmmemctl`) progressively takes memory from the guest OS. From SQL Server's perspective this is indistinguishable from a physical memory shortage.
- Solutions: increase host physical memory, reduce VM memory overcommit ratio, configure memory reservations for the SQL Server VM, or migrate the VM.
- Redgate Monitor's WMI collection may surface VMware guest memory counters in the Activity Graph if VMware Tools is installed â check the Memory category.
