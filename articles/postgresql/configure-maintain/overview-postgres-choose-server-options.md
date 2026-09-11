---
title: Choose the right hosting option in Azure Database for PostgreSQL
description: Compare PostgreSQL hosting options on Azure, including VMs, Flexible Server, Elastic Clusters, and Azure HorizonDB (Preview).
#customer intent: As a user, I want to compare Azure Database for PostgreSQL hosting options so that I can choose the deployment model that fits my workload.
author: varun-dhawan
ms.author: varundhawan
ms.reviewer: maghan
ms.date: 07/08/2026
ms.service: azure-database-postgresql
ms.subservice: configuration
ms.topic: overview
ai-usage: ai-assisted
ms.custom:
  - mvc
---

# Choose the right hosting option in Azure Database for PostgreSQL

Azure offers self-managed PostgreSQL on Azure virtual machines (VMs) and managed PostgreSQL services. Start by deciding whether you need operating system and database-engine control or prefer Azure to manage patching, backups, and high availability.

If you choose a managed service, compare Azure Database for PostgreSQL flexible server, Elastic Clusters, and Azure HorizonDB (Preview). The best fit depends on your workload's scale, distribution model, tenant or service boundaries, read patterns, operational constraints, migration path, and cost model.

## Choose a managed PostgreSQL offering

Use the following comparison to identify the architecture that best matches your workload. Validate the choice against the linked limitations and pricing information before deployment.

| **Decision factor** | **Flexible Server** | **Elastic Clusters** | **Azure HorizonDB (Preview)** |
| --- | --- | --- | --- |
| **Workload fit** | Managed PostgreSQL for workloads that fit a single-server architecture. Match compute and storage to the workload's CPU, memory, IOPS, throughput, and latency requirements. | [Citus-based horizontal sharding](../elastic-clusters/concepts-elastic-clusters.md) for workloads that can distribute data across nodes. | A [fully managed service built on PostgreSQL](../../horizondb/overview.md) for OLTP, AI and vector applications, large read scale-out, and hybrid applications. |
| **Scale and distribution** | Scale compute and storage within a server. Review [operational performance planning](../compute-storage/concepts-optimal-performance.md) and current service limits before assuming that vertical scaling meets future demand. | Add nodes and choose [row-based or schema-based sharding](../elastic-clusters/concepts-elastic-clusters-sharding-models.md). You must distribute tables or schemas to use distributed query processing. | [Scale compute separately from storage and add readable replicas](../../horizondb/configure-maintain/concepts-compute-replicas.md). A cluster supports one writable primary and up to 15 readable replicas. |
| **Tenant and service boundaries** | Use when the application doesn't need managed horizontal sharding across multiple PostgreSQL nodes. | [Row-based sharding](../elastic-clusters/concepts-elastic-clusters-sharding-models.md#row-based-sharding) fits shared-schema multitenancy but requires a distribution column and query changes. [Schema-based sharding](../elastic-clusters/concepts-elastic-clusters-sharding-models.md#schema-based-sharding) fits multitenant and microservice designs that use separate schemas. | Use PostgreSQL databases and schemas for application boundaries. For managed Citus sharding, evaluate [Elastic Clusters](../elastic-clusters/concepts-elastic-clusters.md) instead. |
| **Read, write, and analytical patterns** | Use workload measurements to balance compute and storage for transactional, reporting, and batch activity. | Distributed queries can route to one node or run across shards. [Row-based sharding supports parallel cross-tenant queries](../elastic-clusters/concepts-elastic-clusters-sharding-models.md#sharding-tradeoffs); schema-based sharding doesn't. | Writes use one primary. [Readable replicas scale read throughput and isolate reporting or analytics from transactional writes](../../horizondb/configure-maintain/concepts-compute-replicas.md#scale-out-reads) when reads can tolerate milliseconds of visibility delay. |
| **Material constraints** | Service limits and available configurations vary by compute and storage choice. Check current limits and supported PostgreSQL versions for your planned configuration. | [Elastic Clusters support PostgreSQL 17 and currently have scale-in, major-version-upgrade, extension, performance-feature, and replica limitations](../elastic-clusters/concepts-elastic-clusters-limitations.md). | The service is in Preview. [Scaling compute restarts replicas, temporarily interrupts availability, and drops connections](../../horizondb/configure-maintain/concepts-compute-replicas.md#how-scale-up-works). Review the [current Preview limitations and regional availability](../../horizondb/overview.md) before deployment. |
| **Documented migration path** | Use the documented [`pg_dump` and restore workflow](../migrate/how-to-migrate-using-dump-and-restore.md). Choose utilities that are the same major version as, or newer than, the source and target servers. | [Migrate to or from Elastic Clusters](../elastic-clusters/concepts-elastic-clusters-limitations.md#migrations) with `pg_dump`, `pg_restore`, or `pgcopydb`, and verify extension compatibility first. | The documented [dump-and-restore workflow migrates data to Azure HorizonDB](../../horizondb/migrate/how-to-migrate-dump-restore.md). It requires compatible PostgreSQL utilities and additional handling for roles and permissions. |
| **Pricing model** | Pricing reflects provisioned compute, storage, backup storage beyond the included amount, and applicable network usage. See [Azure Database for PostgreSQL pricing](https://azure.microsoft.com/pricing/details/postgresql/flexible-server/). | Cluster cost reflects the Flexible Server resources provisioned across its nodes. Use [Azure Database for PostgreSQL pricing](https://azure.microsoft.com/pricing/details/postgresql/flexible-server/) to estimate those resources. | [Azure HorizonDB pricing](https://azure.microsoft.com/pricing/details/horizondb/) separately accounts for provisioned compute for the primary and replicas, allocated data and log storage, and applicable backup storage. |

### Choose Flexible Server when

Choose Flexible Server when you want managed PostgreSQL and your workload fits a single-server architecture that you can scale by selecting appropriate compute and storage. Start here when the application doesn't require managed horizontal sharding or Azure HorizonDB (Preview) capabilities.

### Choose Elastic Clusters when

Choose [Elastic Clusters](../elastic-clusters/concepts-elastic-clusters.md) when data and queries have a clear distribution key or schema boundary, and horizontal sharding justifies the added data-model and query-design complexity. The documented [sharding models](../elastic-clusters/concepts-elastic-clusters-sharding-models.md) fit distributed multitenant, microservice, or cross-tenant workloads; Elastic Clusters aren't an automatic upgrade from Flexible Server.

### Choose Azure HorizonDB (Preview) when

Choose [Azure HorizonDB (Preview)](../../horizondb/overview.md) when you need independently scalable compute and storage or its documented AI, vector, and hybrid application capabilities. Its [compute replicas](../../horizondb/configure-maintain/concepts-compute-replicas.md) provide read scale-out and isolate reporting or analytics reads from transactional writes. Confirm that its Preview status, regional availability, single-primary write model, and current limitations fit your requirements.

## Compare self-managed and managed hosting

PostgreSQL on Azure VMs is an infrastructure as a service (IaaS) option. It gives you control over the operating system and database engine, but you manage the VMs and database administration tasks such as patching, recovery, backups, and high-availability design. The managed offerings are platform as a service (PaaS) options that delegate more of those responsibilities to Azure.

| **Attribute** | **PostgreSQL on Azure VMs** | **Azure managed PostgreSQL offerings** |
| --- | --- | --- |
| **Availability SLA** | [Virtual Machine SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines) | [Azure Database for PostgreSQL SLA](https://azure.microsoft.com/support/legal/sla/postgresql) |
| **OS and PostgreSQL patching** | Customer managed | Automatic, with service-specific maintenance controls |
| **High availability** | You architect, implement, test, and maintain high availability. | Built in, with service-specific configuration and behavior |
| **Zone redundancy** | You can deploy Azure VMs in different availability zones. | Available according to the selected service and configuration |
| **Backup and restore** | Customer managed | Built in, with service-specific configuration |
| **Monitoring database operations** | Customer managed | Built-in monitoring and alerting capabilities vary by service |
| **Advanced threat protection** | You build and manage threat protection. | [Microsoft Defender for Cloud integration (Preview)](../security/security-defender-for-cloud.md) is available for Flexible Server. |
| **Disaster recovery** | Customer managed | Capabilities vary by service and configuration |
| **Performance management** | Customer managed | Built-in capabilities vary by service |

## Administration

For many businesses, moving to a cloud service is as much about reducing administration as it is about cost.

With IaaS, Microsoft:

- Administers the underlying infrastructure.
- Provides automated patching for underlying hardware and the operating system.

With PaaS, Microsoft:

- Administers the underlying infrastructure.
- Provides automated patching for underlying hardware, the operating system, and the database engine.
- Manages high availability of the database.
- Automatically performs backups.
- Provides encryption, monitoring, and performance capabilities that vary by service.

With a managed PostgreSQL offering, you continue to administer databases, sign-ins, indexes, queries, auditing, and security. You don't manage the underlying hardware, operating system, or database engine.

With PostgreSQL on Azure VMs, you control the operating system, PostgreSQL server configuration, software installation, patch timing, VM size, disks, and storage configuration. For more information, see [Virtual machine sizes for Azure](/azure/virtual-machines/sizes).

## Total cost of ownership

Total cost of ownership includes both Azure charges and the people and processes required to operate the database. A managed service can reduce administration for patching, backups, and high availability. PostgreSQL on Azure VMs gives you more control, but you pay for the provisioned VMs, storage, backup, monitoring, and log storage, and you operate the database software.

## Billing

The managed offerings don't share one pricing model. Flexible Server pricing reflects its provisioned compute, storage, backup storage, and applicable network usage. Elastic Clusters use Flexible Server resources across cluster nodes. Azure HorizonDB (Preview) has separate charges for provisioned primary and replica compute, data and log storage, and applicable backup storage.

Don't use a static price to compare these architectures. Estimate the workload's required topology and resources with the [Azure pricing calculator](https://azure.microsoft.com/pricing/calculator/), then review the current [Azure Database for PostgreSQL pricing](https://azure.microsoft.com/pricing/details/postgresql/flexible-server/), [Azure HorizonDB pricing](https://azure.microsoft.com/pricing/details/horizondb/), and [Virtual Machines pricing](https://azure.microsoft.com/pricing/details/virtual-machines/).

## Related content

- [What is Azure Database for PostgreSQL?](../overview.md)
- [What is an Elastic Cluster?](../elastic-clusters/concepts-elastic-clusters.md)
- [What is Azure HorizonDB (Preview)?](../../horizondb/overview.md)
