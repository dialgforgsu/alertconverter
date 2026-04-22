# Availability Replicas With Disks in Same Datastore

## Overview
Evaluates to true when multiple AG replicas from the same Availability Group have their VMDKs in the same VMware datastore. A datastore failure would nullify the AG redundancy.

---

## SQL Sentry Advisory Condition

### Original SQL Sentry Query
```sql
-- Uses SQL Sentry's proprietary VM.* and AlwaysOn.* schema tables
SELECT 'Data Store : ' + quotename(ds.name) + N' | WSFC : ' + cr.ClusterName + ' | AG : ' + ag.name ...
FROM vm.datastore AS ds
JOIN vm.virtualMachineVirtualDisk AS vmdk ON ...
JOIN AlwaysOn.AvailabilityReplica AS ar ON ...
GROUP BY cr.ClusterName, ag.Name, ds.Name, ag.GroupId;
```

> **Note:** Uses SQL Sentry's proprietary `VM.*` and `AlwaysOn.*` schema tables — not standard system views.

---

## Redgate Monitor Custom Metric

### NOT AVAILABLE via T-SQL

VMware datastore placement of VM virtual disks is not visible from SQL Server. T-SQL has no access to the VMware storage layer — it cannot determine which datastore a VMDK resides on. This requires VMware vCenter inventory access.

**Why an AG lag proxy is wrong:** Secondary replica lag time is an availability concern but has nothing to do with whether replicas share a datastore. A replica can have zero lag and still be on the same datastore as the primary. These are completely unrelated conditions.

### How to check this out-of-band

- **VMware PowerCLI:**
  ```powershell
  Get-VM | Get-HardDisk | Select @{N='VM';E={$_.Parent.Name}}, Filename
  ```
  The `Filename` format is `[DATASTORE_NAME] path/disk.vmdk` — cross-reference the datastore name against AG replica VM names.
- **vSphere API / vCenter:** Query VM storage placement via the vCenter REST API or vSphere SDK.
- **Redgate Monitor:** Has no native VMware storage integration.

### T-SQL Reference Query (informational only)

Returns AG secondary replica queue sizes as an availability health baseline. Does not detect datastore co-location.

```sql
SELECT
    ag.name                    AS AvailabilityGroupName,
    ar.replica_server_name     AS ReplicaNode,
    drs.log_send_queue_size    AS LogSendQueueKB,
    drs.redo_queue_size        AS RedoQueueKB,
    drs.last_hardened_time     AS LastHardenedTime
FROM sys.dm_hadr_database_replica_states AS drs
INNER JOIN sys.availability_replicas AS ar
    ON drs.replica_id = ar.replica_id
INNER JOIN sys.availability_groups AS ag
    ON ar.group_id = ag.group_id
INNER JOIN sys.dm_hadr_availability_replica_states AS ars
    ON ar.replica_id = ars.replica_id
WHERE ars.role_desc = N'SECONDARY';
```

### Notes
- This is one of the SQL Sentry conditions that genuinely **cannot be replicated** as a Redgate Monitor custom metric.
- Use VMware PowerCLI or vCenter to validate that AG replica VMs have their VMDKs on separate datastores.
- Redgate Monitor's AG pipeline view provides send/redo queue visibility for availability monitoring.