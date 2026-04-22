# High Performance Power Plan Not Enabled

## Overview
If the High Performance power plan is not enabled, Windows may throttle CPU performance via processor P-state frequency scaling, adding unpredictable latency to SQL Server operations. Dedicated SQL Servers should always use the High Performance plan.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
No T-SQL — SQL Sentry uses a WMI query (`SELECT * FROM Win32_PowerPlan WHERE IsActive = True`) to check the active Windows power plan.

---

## Redgate Monitor Custom Metric

### NOT AVAILABLE via T-SQL

The Windows power plan setting is an OS-level configuration not exposed through SQL Server DMVs or system views. There is no T-SQL query that can reliably detect whether the High Performance plan is active.

**Why a clock speed proxy is wrong:** `cpu_ticks / ms_ticks` from `sys.dm_os_sys_info` reflects SQL Server's internal time accounting, not the actual processor clock frequency. P-state scaling occurs at the hardware level below what SQL Server observes — the ratio does not change meaningfully between power plans and cannot detect throttling.

### How to check this out-of-band

- **Definitive check — run on the server:**
  ```cmd
  powercfg /getactivescheme
  ```
  If the output does not contain "High performance" (`8c5e7fda-e8bf-4a96-9a85-a6e23a8c635c`), the plan is wrong.

- **Enforce via Group Policy:** `Computer Configuration → Administrative Templates → System → Power Management → Active Power Plan`

- **Enforce via script (run as Administrator):**
  ```cmd
  powercfg /setactive 8c5e7fda-e8bf-4a96-9a85-a6e23a8c635c
  ```

- **Redgate Monitor:** The built-in WMI collection may surface power plan information in the Server Overview — check the Hardware/OS section.

### T-SQL Reference Query (informational only)

Returns SQL Server CPU utilisation from the ring buffer as a performance baseline. Does not detect power plan status.

```sql
SELECT TOP 1
    record.value('(./Record/SchedulerMonitorEvent/SystemHealth/ProcessUtilization)[1]', 'INT')
        AS SqlServerCpuPercent,
    record.value('(./Record/SchedulerMonitorEvent/SystemHealth/SystemIdle)[1]', 'INT')
        AS SystemIdlePct
FROM (
    SELECT CAST(record AS XML) AS record
    FROM sys.dm_os_ring_buffers
    WHERE ring_buffer_type = N'RING_BUFFER_SCHEDULER_MONITOR'
      AND record LIKE N'%<SystemHealth>%'
) AS rb
ORDER BY record.value('(./Record/@id)[1]', 'BIGINT') DESC;
```

### Notes
- This is one of the SQL Sentry conditions that genuinely **cannot be replicated** as a Redgate Monitor custom metric.
- The High Performance plan prevents processor P-state and C-state transitions that add microsecond-level latency compounding on high-frequency OLTP workloads.
- Verify the power plan as part of your SQL Server build checklist before the server goes into production.