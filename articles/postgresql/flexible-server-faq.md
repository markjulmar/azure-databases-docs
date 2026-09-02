---
title: Flexible Server Billing, Backup, and Scaling FAQ
description: Get answers about Azure Database for PostgreSQL flexible server charges, backup storage growth, scaling options, and restart effects.
author: varun-dhawan
ms.author: varundhawan
ms.date: 09/02/2026
ms.service: azure-database-postgresql
ms.topic: faq
ai-usage: ai-generated
---

# Frequently asked questions about Azure Database for PostgreSQL flexible server

This FAQ helps developers, database administrators, and decision-makers understand common Azure Database for PostgreSQL flexible server billing, backup storage, and scaling behavior. Use the answers and linked guidance to evaluate charges and plan capacity changes.

## Cost and billing

The following questions explain ongoing charges and backup storage growth.

### Why am I still being charged for my Azure Database for PostgreSQL flexible server?

A running Azure Database for PostgreSQL flexible server accrues compute charges, while provisioned storage and excess backup storage are billed independently of database activity. Stopping the server stops compute billing, but storage and backup storage can continue to incur charges. Review [cost optimization guidance](configure-maintain/how-to-cost-optimization.md) and [current Azure Database for PostgreSQL pricing](https://azure.microsoft.com/pricing/details/postgresql/) for details.

### Why is my Azure Database for PostgreSQL flexible server's backup storage growing to terabytes despite a small database size?

Backup storage can exceed the visible database size because backup usage includes retained snapshots and continuously archived write-ahead log (WAL) files, not only current database data. Heavy transaction activity and the backup retention period can increase retained WAL, geo-redundant backup affects billed backup usage, and an unused logical replication slot can separately cause WAL to accumulate on primary storage. The service includes a backup storage allowance based on provisioned server storage and bills usage over that allowance; see [backup and restore concepts](backup-restore/concepts-backup-restore.md) and [flexible server limits](configure-maintain/concepts-limits.md).

## Scaling

The following question explains scaling options and their availability effects.

### How do I scale compute and storage for Azure Database for PostgreSQL flexible server, and is there downtime?

You can scale compute and storage independently for a running Azure Database for PostgreSQL flexible server through the Azure portal, Azure CLI, or Azure REST API. Changing compute requires a server restart and can interrupt connections; current storage scale-up also requires a restart, and storage can increase but not decrease. For details, see [scaling resources](scale/concepts-scaling-resources.md), [scale compute](scale/how-to-scale-compute.md), [scale storage size](scale/how-to-scale-storage-size.md), and the [`az postgres flexible-server update` reference](/cli/azure/postgres/flexible-server#az-postgres-flexible-server-update).

## Related content

- [How to optimize costs](configure-maintain/how-to-cost-optimization.md)
- [Backup and restore in Azure Database for PostgreSQL flexible server](backup-restore/concepts-backup-restore.md)
- [Limits in Azure Database for PostgreSQL flexible server](configure-maintain/concepts-limits.md)
- [Scaling resources in Azure Database for PostgreSQL flexible server](scale/concepts-scaling-resources.md)
