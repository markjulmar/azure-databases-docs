---
title: Troubleshoot Storage Growth and Full-Disk Conditions
description: Diagnose storage growth and full-disk conditions, recover writes safely, resolve the storage consumer, and prevent recurrence.
author: markjulmar
ms.author: markjulmar
ms.date: 09/10/2026
ms.service: azure-database-postgresql
ms.subservice: performance
ms.topic: troubleshooting
ai-usage: ai-generated
---

# Troubleshoot storage growth and full-disk conditions in Azure Database for PostgreSQL

A full disk can cause an Azure Database for PostgreSQL flexible server to reject writes, affect backups or write-ahead log (WAL) archiving, and become unavailable. The service switches a server to read-only mode when storage usage reaches 95 percent or available storage falls below 5 GiB. You might first notice write failures, a storage alert, or a rapid rise in **Storage percent** or **Storage Used**.

Use this article to confirm storage pressure, identify the storage consumer, apply the least irreversible fix, and verify recovery. Provisioning quota, region, availability zone, and SKU errors are different problems; see [Resolve capacity errors](how-to-resolve-capacity-errors.md). Elastic clusters are also outside this article's scope because their diagnostics must run across nodes. For node-aware autovacuum guidance, see [Autovacuum tuning for elastic clusters](how-to-autovacuum-tuning-elastic-clusters.md).

## Prerequisites

- Access to the flexible server in the Azure portal, with permission to view metrics and alerts.
- A database connection that can query PostgreSQL catalog views.
- Permission to change or remove the object responsible for growth when remediation requires it.
- An action group or another notification target if you configure alerts.

<!-- TODO: [C4] SME: Specify and test the Azure/PostgreSQL roles or grants required to run pg_ls_waldir(), inspect all rows in pg_stat_activity and pg_replication_slots, call pg_drop_replication_slot(), and terminate another backend. -->

## Recover a server that's already read-only

Use a read-write session only to reclaim enough space for controlled recovery. Don't resume the normal write workload while storage remains near the full-disk threshold.

1. In Azure Monitor, check **Storage percent**, **Storage Used**, **Storage Free**, **Transaction Log Storage Used**, and **Database Is Alive**. If storage isn't under pressure, use a different troubleshooting path.
1. Connect to the affected database and enable writes for the current session:

   ```sql
   SET SESSION CHARACTERISTICS AS TRANSACTION READ WRITE;
   ```

1. Delete only data that you've confirmed is no longer required. Monitor storage metrics during the operation, and stop broad write activity if storage continues to rise.
1. Confirm that **Storage percent** falls or **Storage Free** rises. Then test an application write and confirm that **Database Is Alive** reports `1`.

The command changes transaction characteristics for the session. It doesn't add storage, identify the cause, or make an unrestricted workload safe. Diagnose the cause before you scale storage because a storage increase can't be reversed.

## Troubleshooting checklist

Follow this order so that you collect non-destructive evidence before dropping a slot, terminating a backend, deleting data, or increasing storage.

1. Confirm that **Storage percent** or **Storage Used** is rising and **Storage Free** is falling.
1. Compare **Transaction Log Storage Used** with **Database Size**. A rise in transaction-log storage points toward WAL retention; database-size growth points toward relations or ordinary data.
1. Check replication slots and replication lag.
1. Check dead tuples, autovacuum activity, long-running transactions, and prepared transactions.
1. Check temporary-file metrics and the largest relations.
1. Apply only the remediation for the cause you confirm, and then repeat the relevant check.

To measure the current WAL directory, run the following reviewable PostgreSQL query:

```sql
SELECT pg_size_pretty(sum(size)) AS wal_directory_size
FROM pg_ls_waldir();
```

<!-- TODO: SME review required [TR3]: Test the pg_ls_waldir() aggregate on supported flexible-server versions and document its required role or grants. -->
<!-- TODO: [TR3] SME: Add validated representative output and explain that WAL-directory size alone doesn't identify whether a slot, archiving, or a lagging standby retains the WAL. -->

## Causes and solutions

The following checks separate the major storage consumers. Treat sample rows as interpretation examples, not expected values for your server.

### Inactive logical replication slot

An unconsumed logical slot can retain WAL. The `active` value shows whether a consumer is connected; retained WAL that grows while `active` is `false` identifies a slot that needs investigation.

```sql
SELECT slot_name,
       slot_type,
       database,
       active,
       restart_lsn,
       confirmed_flush_lsn,
       pg_size_pretty(
           pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn)
       ) AS retained_wal
FROM pg_replication_slots
ORDER BY pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn) DESC;
```

<!-- TODO: SME review required [TR3]: Test the retained-WAL expression on supported flexible-server versions, confirm role requirements, and replace the illustrative interpretation if its shape differs. -->

For example, a row with `active = false` and a large or increasing `retained_wal` value indicates that the slot has no connected consumer and is retaining WAL. Confirm with the owner of the replication workload before changing the slot.

1. If the slot is required, restore its consumer and continuously consume changes.
1. If the slot is no longer required, drop it:

   ```sql
   SELECT pg_drop_replication_slot('unused_slot_name');
   ```

1. Confirm that the slot is active and consumed, or that the unused slot no longer appears in `pg_replication_slots`. Verify that **Transaction Log Storage Used** stops growing or decreases.

Azure can automatically drop an unused logical slot under high storage pressure when its subscriber is unavailable, which releases accumulated WAL. Don't rely on that behavior as remediation; consume required slots and remove unused slots promptly.

### WAL archiving or lagging physical standby

WAL growth can also accompany delayed archiving or replication lag. Compare **Transaction Log Storage Used**, **Max Logical Replication Lag**, and **Max Physical Replication Lag** to the incident timeline.

<!-- TODO: [TR2, TR3] SME: Add tested catalog queries, decisive fields, and representative output that distinguish failed or delayed archiving from WAL retained for a lagging physical standby. -->
<!-- TODO: [TR5] SME: Add supported cause-specific actions and post-action checks for failed archiving and a lagging physical standby, including when to repair connectivity, remove a replica, or escalate. -->

1. Preserve the WAL and replication metrics for the incident window.
1. Don't use checkpoint tuning as a substitute for diagnosing archiving or standby retention.
1. Escalate if WAL archiving remains affected, the server is unavailable, or transaction-log storage doesn't fall after the supported cause-specific repair.

### Table or index bloat and lagging autovacuum

Frequent updates and deletes create dead tuples. If autovacuum can't keep up, PostgreSQL can reuse space after vacuuming, but ordinary `VACUUM` doesn't necessarily shrink physical files.

```sql
SELECT schemaname,
       relname,
       n_live_tup,
       n_dead_tup,
       round(100.0 * n_dead_tup /
             greatest(n_live_tup + n_dead_tup, 1), 2) AS dead_percent,
       last_autovacuum,
       last_autoanalyze
FROM pg_stat_all_tables
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY n_dead_tup DESC
LIMIT 20;
```

A high `n_dead_tup` or `dead_percent` value with old or empty autovacuum timestamps points toward lagging cleanup. Compare these results with **Bloat Percent** and **Estimated Dead Rows User Tables**.

1. Remove blockers, and keep autovacuum enabled so that it can make dead-tuple space reusable.
1. Use [Autovacuum tuning](how-to-autovacuum-tuning.md) when autovacuum isn't keeping up.
1. If you must return physical space to the storage layer, review [Full vacuum using pg_repack](how-to-perform-fullvacuum-pg-repack.md) and its prerequisites instead of assuming that `VACUUM` shrinks files.
1. Confirm that blocker queries disappear, dead tuples or bloat decrease, and vacuum or analyze timestamps advance.

### Long-running or prepared transactions

Long-running and uncommitted prepared transactions can prevent autovacuum from removing dead tuples.

```sql
SELECT pid,
       age(backend_xid) AS age_in_xids,
       now() - xact_start AS xact_age,
       state,
       query
FROM pg_stat_activity
WHERE state <> 'idle'
ORDER BY age(backend_xid) DESC NULLS LAST
LIMIT 10;

SELECT gid, prepared, owner, database, transaction
FROM pg_prepared_xacts
ORDER BY age(transaction) DESC;
```

For example, an old `age_in_xids` with a long `xact_age`, or an old row in `pg_prepared_xacts`, identifies a transaction that can block cleanup.

1. Confirm the session or prepared transaction with its workload owner.
1. Commit or roll back a prepared transaction. If appropriate for the workload, terminate the identified backend rather than unrelated sessions.
1. Rerun both queries and the dead-tuple check. Confirm that the blocker is gone and autovacuum timestamps advance.

### Temporary files from query spills

Rising **Temporary Files** (`temp_files`) and **Temporary Files Size** (`temp_bytes`) metrics identify databases where queries write temporary data. Spilled sorts and hashes can contribute to this pattern.

<!-- TODO: [TR2, TR3, TR5] SME: Add a runnable flexible-server SQL query for temporary files, explain whether its counters are cumulative and reset-aware, provide validated representative output, and document a safe query-specific remediation and verification check. -->
<!-- TODO: SME review required [TR2, TR5]: Confirm the exact diagnostic view and safe scope for query-specific work_mem tuning before recommending it for production spills. -->

1. Use the portal **Temporary files** troubleshooting guide to correlate storage, temporary files, workload, and queries.
1. Identify and correct the responsible query or workload before changing memory settings broadly.
1. Verify that the temporary-file growth rate falls while the workload runs.

### Ordinary data growth

Ordinary growth is the remaining pattern when **Database Size** and **Storage Used** rise without matching WAL, temporary-file, or bloat evidence.

<!-- TODO: [TR2, TR3, TR5] SME: Add a tested query that reports database, schema, table, index, and total relation sizes; include representative output that confirms ordinary growth and safe reclamation or capacity actions. -->

1. Confirm that the data is required and review retention needs before deleting rows or objects.
1. Remove only unneeded data, or plan additional capacity for required growth.
1. Verify that **Database Size** and **Storage Used** follow the expected post-change trend.

### Storage increase as a last resort

Increase storage only when cause-specific remediation can't create enough headroom or the retained data is required.

1. For Premium SSD, consider storage autogrow for prevention or increase provisioned storage.
1. For Premium SSD v2, adjust capacity separately from IOPS and throughput. Storage autogrow isn't available for this disk type.
1. Confirm the planned capacity and quota before applying the change. You can scale storage up for either disk type, but you can't scale it down later.
1. After scaling, continue the root-cause repair so that the same consumer doesn't fill the larger disk.

## Verify recovery and prevent recurrence

Recovery requires both healthy metrics and a successful write. Continue monitoring until the repaired consumer remains stable.

1. Confirm that **Storage percent**, **Storage Used**, **Storage Free**, and **Transaction Log Storage Used** move in the expected direction.
1. Confirm that **Database Is Alive** reports `1` and an application write succeeds.
1. Configure an alert on **Storage percent** or **Storage Used** before the 95-percent full-disk threshold. For example, the alerting guidance uses an 80-percent static threshold as a proactive starting point.
1. Add alerts or monitoring for **Maximum Used Transaction IDs**, replication lag, transaction-log storage, temporary files, bloat, dead rows, and oldest transaction indicators when those risks apply.
1. Escalate if storage remains near the read-only threshold, backups or WAL archiving remain affected, the server is unavailable, or the required increase exceeds quota or regional capacity.

For the portal procedure, see [Configure alerts on metrics](../monitor/how-to-alert-on-metrics.md).

## Related content

- [Storage in Azure Database for PostgreSQL flexible server](../compute-storage/concepts-storage.md)
- [Logical replication and logical decoding](../configure-maintain/concepts-logical.md)
- [Monitor Azure Database for PostgreSQL with metrics and logs](../monitor/concepts-monitoring.md)
