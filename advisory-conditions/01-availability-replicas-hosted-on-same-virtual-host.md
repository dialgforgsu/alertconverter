# Availability Replicas Hosted on Same Virtual Host

## Overview
Evaluates to true when multiple Availability Replicas from the same Availability Group are hosted on the same VMware/Hyper-V host. If the host fails, the AG redundancy is nullified.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
SELECT
  'Host : ' + QUOTENAME(HS1.Name) + ' | WSFC : ' + CR.ClusterName + ' | AG : ' + AG1.Name + ' - ' +
   STUFF((SELECT N', ' + QUOTENAME(NodeName) + '(' + CASE Role WHEN 1 THEN 'P' ELSE 'S' END + ')'
    FROM AlwaysOn.AvailabilityReplica AS AR2
    INNER JOIN VM.VirtualMachine AS VM2 ON AR2.NodeName = VM2.Name
    WHERE AR2.GroupID = AR1.GroupID AND VM2.HostSystemID = VM1.HostSystemID
    ORDER BY Role, NodeName FOR XML PATH(N''), TYPE).value(N'.[1]', N'nvarchar(max)'), 1, 2, N'') AS ConditionKey
 ,COUNT(VM1.Name) AS ReplicaCount
FROM VM.HostSystem HS1
INNER JOIN VM.VirtualMachine AS VM1 ON HS1.ID = VM1.HostSystemID
INNER JOIN AlwaysOn.AvailabilityReplica AR1 ON AR1.NodeName = VM1.Name
INNER JOIN AlwaysOn.AvailabilityGroup AS AG1 ON AG1.GroupID = AR1.GroupID
INNER JOIN AlwaysOn.ClusterReference CR ON AR1.EventSourceConnectionID = CR.EventSourceConnectionID
GROUP BY CR.ClusterName, AR1.GroupID, AG1.Name, VM1.HostSystemID, HS1.Name
HAVING COUNT(VM1.Name) > 1;
```

> **Note:** Uses SQL Sentry's proprietary `VM.*` and `AlwaysOn.*` schema tables populated by its hypervisor integration — not standard SQL Server system views.

---

## Redgate Monitor Custom Metric

### NOT AVAILABLE via T-SQL

VM host co-location cannot be determined from SQL Server DMVs or system views. This condition requires hypervisor-level inventory data (VMware vCenter or Hyper-V host records) to map each AG replica's node name to its physical host.

**Why a replica count proxy is wrong:** The number of synchronous-commit replicas has no relationship to where those replicas are physically hosted. A cluster with two replicas on the same host looks identical to one spread across separate hosts from SQL Server's perspective.

### How to check this out-of-band

- **VMware:** Use VMware PowerCLI — `Get-VM | Select Name, VMHost` — cross-reference AG replica server names against VM-to-host placement.
- **Hyper-V:** Run `Get-VM -ComputerName <each_host>` to enumerate VMs per host.
- **Both:** A CMDB or configuration management system that tracks server-to-host placement is the appropriate solution.

### T-SQL Reference Query (informational only)

Returns AG replica node names for cross-referencing against hypervisor host inventory. Does not return a scalar alert value.

```sql
SELECT
    ag.name                          AS AvailabilityGroupName,
    ar.replica_server_name           AS ReplicaNode,
    ar.availability_mode_desc        AS SyncMode,
    ars.role_desc                    AS CurrentRole
FROM sys.availability_groups AS ag
INNER JOIN sys.availability_replicas AS ar
    ON ag.group_id = ar.group_id
LEFT JOIN sys.dm_hadr_availability_replica_states AS ars
    ON ar.replica_id = ars.replica_id
ORDER BY ag.name, ar.replica_server_name;
```

### Notes
- This is one of the SQL Sentry conditions that genuinely **cannot be replicated** as a Redgate Monitor custom metric.
- Redgate Monitor's AG pipeline view shows replica roles and sync state but has no visibility into hypervisor host placement.
- For proactive placement monitoring, schedule a PowerCLI or Hyper-V script that validates AG replicas are spread across separate hosts.