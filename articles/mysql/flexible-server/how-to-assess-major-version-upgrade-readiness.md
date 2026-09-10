---
title: Assess MySQL 8.0 to 8.4 Upgrade Readiness
description: Assess an Azure Database for MySQL Flexible Server workload and document a go, remediate, or delay decision before upgrading to MySQL 8.4.
author: hariramt
ms.author: hariramt
ms.reviewer: maghan
ms.date: 09/10/2026
ms.service: azure-database-mysql
ms.subservice: flexible-server
ms.topic: how-to
ai-usage: ai-generated
ms.custom:
  - devx-track-azurecli
---

# Assess readiness for a MySQL 8.0 to 8.4 upgrade

A readiness assessment determines whether an Azure Database for MySQL Flexible Server workload can move from MySQL 8.0 to 8.4 without unacceptable compatibility, performance, or recovery risk. Run the assessment before you open the production maintenance window so that you can remediate issues without affecting production.

This article helps you inventory the source server, validate a nonproduction copy on MySQL 8.4, compare workload behavior, and document a go, remediate, or delay decision. It doesn't replace the [major version upgrade procedure](how-to-upgrade.md).

## Prerequisites

- An Azure Database for MySQL Flexible Server instance that runs MySQL 8.0.
- Access to the Azure portal and, for command-line checks, Azure CLI or Azure Cloud Shell.
- The subscription ID, resource group, server name, and credentials required to inspect the server and its databases.
- Permission to create a read replica or point-in-time restored server and to take an on-demand backup.
- A representative set of application connections, queries, reads, writes, stored routines, and workload metrics.
- Access to MySQL Shell and the MySQL Upgrade Checker utility.

## Confirm the target and support timeline

Confirm that this assessment applies to your upgrade path before you create validation resources.

1. Verify that the source is MySQL 8.0 and that the intended target is MySQL 8.4. Azure Database for MySQL lists both versions as generally available. Direct upgrades that skip a major version aren't supported; this article doesn't provide a MySQL 5.7-to-8.4 path.
1. Record the lifecycle dates in the readiness record. Azure Standard Support for MySQL 8.0 ends on December 31, 2026. Extended Support and automatic enrollment start on January 1, 2027. For current lifecycle details, see the [Azure Database for MySQL version support policy](../concepts-version-policy.md).

<!-- TODO: [C3] SME: resolve the inconsistent Extended Support charging and grace-period wording in the version policy before adding an exact billing-start statement. -->

## Inventory the source server and workload

Capture the configuration and workload inputs that the MySQL 8.4 validation target must reproduce. Store the inventory findings with the readiness record.

1. Record the subscription, resource group, server name, MySQL version, compute tier and SKU, high-availability configuration, backup retention, authentication configuration, and replica topology.

   List the server parameters. Replace `<resource-group>` with the resource group that contains the server, and replace `<server-name>` with the server name.

   ```azurecli
   az mysql flexible-server parameter list \
     --resource-group <resource-group> \
     --server-name <server-name>
   ```

   Record each relevant parameter name and value so that you can reproduce or remediate the configuration before validation.

   <!-- TODO: [HT3] SME: add verified read-only Azure CLI commands, required flags, and relevant output fields for engine version, SKU and tier, high availability, replicas, and backup retention. -->

1. Inventory database accounts and their authentication plugins.

   Run the following read-only query:

   ```sql
   SELECT user, host, plugin
   FROM mysql.user;
   ```

   Map each returned `user` and `host` combination to the application, authentication `plugin`, and client driver it uses.

   <!-- TODO: [HT3] SME: confirm any Azure-specific filtering and the least privilege needed to run the account-plugin inventory query on Azure Database for MySQL Flexible Server. -->

1. Record schemas, stored routines, client and connector versions, representative queries, peak and typical workload periods, and application-level latency, throughput, and error measurements. Define workload-specific acceptance thresholds instead of applying a universal percentage.

## Choose a nonproduction validation target

Use either a read replica or a point-in-time restored server to test MySQL 8.4. Both options create persistent, billable Azure resources.

1. Choose a read replica when you need continuing replication from production and want to evaluate a possible replica-based cutover. Preserve the same Microsoft Entra configuration across replica partners. Don't change Microsoft Entra authentication settings while the primary and replica run different MySQL versions.
1. Choose a point-in-time restored server when you need an isolated copy at a known restore point. A restore creates a new server with a unique name. Before testing, reproduce the required networking, permissions, alerts, and application connectivity.
1. Create and upgrade the selected validation target by following the [major version upgrade procedure](how-to-upgrade.md). Don't run the production upgrade yet.
1. Confirm in the Azure portal that the validation server runs MySQL 8.4. If you use a replica, confirm that replication remains healthy before treating replica cutover as an option:

   ```sql
   SHOW SLAVE STATUS\G
   ```

   `Slave_IO_Running` and `Slave_SQL_Running` must show `Yes`, and `Seconds_Behind_Master` must be `0` at the cutover checkpoint.

> [!WARNING]
> Stopping replication or promoting a replica is irreversible. The replica becomes an independent read/write server and can't become a replica again. Don't promote it only to perform readiness tests.

## Run the compatibility assessment

Azure online validation isn't currently supported for the MySQL 8.0-to-8.4 path. Run the MySQL Shell Upgrade Checker against the source, review findings, remediate them, and rerun the checker.

<!-- TODO: SME (C4, HT2, and HT3): add the current MySQL Shell Upgrade Checker invocation for an Azure-hosted MySQL 8.0 source targeting 8.4, including the target-version option, minimum MySQL Shell version, Azure connection form, required privileges, severity meanings, and exit-status interpretation. -->

1. Run the verified Upgrade Checker command from a client that can connect to the source server.
1. Treat findings labeled as errors as blockers. Review warnings and notices at their displayed severity, assign an owner and disposition to each finding, and don't infer that every warning blocks the upgrade.
1. Remediate removed or incompatible configuration and schema objects. Then rerun the same assessment until no unresolved error remains.

<!-- TODO: SME review required (HT2 and HT5): verify that the removed-variable, deprecated-variable, and changed-default checks described by the Upgrade Checker message catalog apply to the installed MySQL Shell version and the Azure MySQL 8.0-to-8.4 path. -->

## Test MySQL 8.4 compatibility changes

Run compatibility tests on the MySQL 8.4 validation target with the same accounts, clients, and data patterns that production uses.

1. Test every application's production driver and authentication path. In Azure Database for MySQL 8.4, `caching_sha2_password` is the default for new accounts, while existing `mysql_native_password` accounts remain supported. Don't assume a connector version is compatible based only on its age; require a successful connection and representative operation.
1. Review Upgrade Checker findings for removed or deprecated parameters, options, and `sql_mode` values. Remediate applicable errors, and rerun the checker.
1. Check for prepared XA transactions on the source:

   ```sql
   XA RECOVER;
   ```

   Resolve every returned transaction before the production upgrade. If the transaction must be rolled back, replace `<xid>` with its transaction ID:

   ```sql
   XA ROLLBACK '<xid>';
   ```

1. Check schema objects and application queries for identifiers that become reserved in MySQL 8.4.

   <!-- TODO: SME (HT2 and HT3): add the exact words newly reserved between MySQL 8.0 and 8.4 and a verified query or Upgrade Checker example that detects unquoted uses. -->

1. Replay writes that use timestamp literals with fractional seconds and time-zone offsets. Compare the stored values with the application input because this pattern has a documented data-consistency risk. Use the [major version upgrade FAQ](how-to-upgrade-faq.md) for the detailed mitigation.
1. Replay representative indexed-string queries that use `IN()` and queries with large `IN()` lists. Include the same value sizes, list sizes, collations, and parameters that production uses. The upgrade FAQ documents both patterns and their mitigations.

## Establish and compare the workload baseline

Compare MySQL 8.0 and 8.4 under similar data volume, load, cache conditions, and test duration. Keep the same workload and measurement definitions on both servers.

<!-- TODO: SME (HT3 and HT4): add a tested Performance Schema or `sys` query that captures digest text, execution count, latency, rows examined, and errors comparably on Azure MySQL 8.0 and 8.4, including least privileges and output interpretation. -->

1. Capture the agreed application and database metrics on MySQL 8.0, replay the representative workload on MySQL 8.4, and capture the same metrics.
1. Compare plans for critical queries and both documented `IN()` patterns. Replace the sample identifiers and values with a representative production query:

   ```sql
   -- Replace these illustrative identifiers and values with a representative production query.
   EXPLAIN
   SELECT column_name
   FROM table_name
   WHERE indexed_string_column IN ('value1', 'value2');
   ```

   Investigate a plan that changes from `range` access on MySQL 8.0 to `ALL` or `index` on MySQL 8.4. `ALL` indicates a full table scan, and `index` indicates a full index scan.
1. If you use `EXPLAIN ANALYZE`, run it only on the nonproduction validation target because it executes the statement. Compare actual time, rows, and loops as well as the access method.

   <!-- TODO: SME review required [HT4]: verify `EXPLAIN ANALYZE` syntax, reported fields, and the safety qualification for Azure MySQL 8.0 and 8.4. -->

1. Verify application connections, authentication, critical reads and writes, stored routines, timestamp results, and error handling. Record each result against its workload-defined threshold.

## Record the readiness and recovery decision

Finish the assessment with a decision that an approver can trace to the validation evidence.

1. Choose **Go** only when the validation target runs MySQL 8.4, Upgrade Checker errors are resolved, required application tests pass, measured results meet the workload-defined thresholds, and a usable pre-upgrade backup is available.
1. Choose **Remediate** when an issue has an owner and a supported correction that you can retest before the maintenance window. Choose **Delay** when an unresolved blocker, unacceptable regression, missing recovery prerequisite, or unowned risk remains.
1. Record the selected cutover approach. An in-place upgrade retains the server identity and connection string but has downtime and can't be reversed. A replica- or restore-based cutover requires application connection changes but lets you validate the new server before redirecting clients.
1. Record the expected maintenance duration from the validation run, unresolved findings and owners, acceptance thresholds, the backup checkpoint, and the exact point at which the team stops or proceeds.
1. Rehearse recovery by restoring a pre-upgrade automated or on-demand backup within its retention period to a new server, validating networking and permissions, and documenting how to redirect clients. Recovery doesn't downgrade or reverse the upgraded server.

<!-- TODO: SME review required [C3]: normalize the requested "Business Critical" tier terminology against the current service-tier name before publication. The draft distinguishes Burstable from non-Burstable planning without asserting that Business Critical and Memory-Optimized are interchangeable. -->

For a Burstable server, include the temporary move to General Purpose compute in the duration and cost plan. If an upgrade fails, the compute tier doesn't automatically return to the previous Burstable SKU. General Purpose and other non-Burstable servers don't use this temporary Burstable transition.

## Clean up validation resources

Remove validation resources only after you retain the evidence needed for approval or remediation.

1. Export or retain the readiness record, test results, plans, and metric comparisons according to your organization's requirements.
1. Delete the validation replica or restored server when no further testing or cutover use remains, and revert any temporary validation-only settings, including autoscaled IOPS when applicable. Don't delete the production source as cleanup.

## Next step

> [!div class="nextstepaction"]
> [Perform the major version upgrade](how-to-upgrade.md)
