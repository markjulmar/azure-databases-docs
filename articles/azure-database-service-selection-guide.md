---
title: Choose the Right Azure Database Service for Your Workload
description: Compare Azure database services by workload, data model, scale, latency, consistency, operations, migration needs, and cost.
author: markjulmar
ms.author: markjulmar
ms.date: 07/23/2026
ms.service: azure
ms.topic: overview
---

# What is the right Azure database service for your workload?

The right Azure data service depends on how your workload stores, relates, and accesses data. Start with the workload model and access patterns, apply nonfunctional requirements, and then create a shortlist. This approach helps you avoid selecting a familiar database engine before you understand the problem it must solve.

This article compares managed Azure services at selection-framework depth. After you have a shortlist, use the linked product documentation to validate current capabilities, regional availability, service limits, pricing, and migration paths.

## Why use a workload-first selection process

A workload-first process makes the reasons for a service choice explicit. It helps development, architecture, operations, security, and finance teams evaluate the same requirements and revisit the decision when those requirements change.

The process also supports **polyglot persistence**: using more than one data store when separate parts of an application have different access patterns. For example, a relational database can remain the system of record while Azure Managed Redis accelerates repeated reads and an analytical service processes telemetry. Don't force every data model into one service solely to simplify the product list.

## Azure data service families

Classify each distinct workload before comparing products:

- **Online transaction processing (OLTP)** handles frequent inserts, updates, and point queries. Relational candidates include Azure SQL Database, Azure SQL Managed Instance, Azure Database for PostgreSQL flexible server, and Azure Database for MySQL flexible server.
- **Document, key-value, graph, or vector operations** often need flexible models, partitioning, or geographic distribution. Azure Cosmos DB provides several APIs for these operational patterns.
- **Caching** keeps frequently accessed or transient data close to an application. Azure Managed Redis is the current managed Redis service for new designs.
- **Online analytical processing (OLAP), near-real-time analytics, and time-series analysis** scan or aggregate large event and historical datasets. Azure Data Explorer and Microsoft Fabric Real-Time Intelligence are analytical candidates rather than OLTP database replacements.
- **Migration-driven workloads** begin with a source engine and compatibility requirements. Source compatibility can narrow the shortlist, but it doesn't replace validation against the target workload.

The following table compares the primary managed database candidates. Capabilities vary by deployment option, API, tier, region, and configuration.

| Service | Strong fit | Scale, latency, and consistency considerations | Management and cost posture | Consider another service when |
| --- | --- | --- | --- | --- |
| **Azure SQL Database** | Cloud-native relational applications that need SQL Server capabilities at the database level | Managed relational consistency; scale options depend on the deployment model and tier | High platform management; cost is driven by compute, storage, service tier, redundancy, and usage model | The workload requires broad instance-level SQL Server compatibility, an open-source engine, or a nonrelational partition model |
| **Azure SQL Managed Instance** | SQL Server migrations that require more instance-level compatibility and minimal application change | Managed relational consistency with instance-oriented deployment and scaling choices | High platform management; its instance boundary can carry a different cost profile than individual databases | The application is cloud-native and database-scoped, or migration assessment shows that instance compatibility isn't needed |
| **Azure Database for PostgreSQL flexible server** | Applications standardized on PostgreSQL and its ecosystem | Relational transactions with vertical scaling and supported read-scaling options | Managed backups, patching, and availability; cost depends on compute, storage, availability, and consumption commitments | The workload depends on another engine, needs a nonrelational distribution model, or requires control available only with self-management |
| **Azure Database for MySQL flexible server** | Applications standardized on MySQL and its ecosystem | Relational transactions with compute, storage, and read-scaling choices | Managed backups, patching, and availability; cost depends on compute, storage, availability, and consumption commitments | The workload needs another engine's semantics, nonrelational partitioning, or infrastructure-level control |
| **Azure Cosmos DB** | Operational document, key-value, graph, vector, or compatible API workloads that need partitioning and geographic distribution | Multiple consistency choices; latency and throughput depend on partition design, regions, API, and capacity model | Fully managed; cost is influenced by provisioned or consumed throughput, storage, regions, and features | The workload depends on relational joins or multirow relational transactions, or its access pattern doesn't provide a sound partition key |
| **Azure Managed Redis** | Cache, session, messaging, and other in-memory, low-latency access patterns | In-memory performance and scaling depend on tier, clustering, data size, and persistence choices | Managed Redis operations; cost is driven largely by memory, tier, replicas, clustering, and networking | Data must live in a durable system of record, the workload needs relational queries, or the working set isn't suitable for memory |

## Database decision framework

Gather these inputs for each workload boundary:

- **Transactions and queries**: Read-to-write ratio, transaction scope, joins, aggregations, scans, point lookups, and query predictability.
- **Data shape**: Schema stability, relationships, document boundaries, keys, graph traversal, vector search, and time ordering.
- **Behavior targets**: Required consistency, latency, throughput, data volume, growth, and peak-to-average demand.
- **Resilience**: Availability objectives, recovery needs, and geographic distribution.
- **Operations**: Desired management responsibility, database-engine control, monitoring, maintenance, and team skills.
- **Constraints**: Security and governance policy, permitted Azure regions, data residency, migration source, compatibility, and cost posture.

Use the resulting profile to branch toward candidates:

- For **relational OLTP**, choose an engine family first. Compare Azure SQL Database with Azure SQL Managed Instance for SQL Server workloads. Compare [Azure Database for PostgreSQL flexible server](postgresql/overview.md) and [Azure Database for MySQL flexible server](mysql/flexible-server/overview.md) when engine compatibility or ecosystem is a requirement.
- For **document, key-value, graph, or vector operational data**, evaluate the matching Azure Cosmos DB API, partition key, consistency model, and distribution needs.
- For **cache acceleration or transient in-memory data**, add Azure Managed Redis alongside the durable system of record. Don't use a cache as the only durable database.
- For **OLAP, telemetry, streaming, or time-series analysis**, compare [Azure Data Explorer](/azure/data-explorer/data-explorer-overview) and [Microsoft Fabric Real-Time Intelligence](/fabric/real-time-intelligence/overview). Keep transactional writes in an OLTP service when the analytical system isn't designed to own those transactions.
- For a **migration**, use the source engine and required compatibility to create an initial target list. Then reassess query behavior, scale, operations, and cost so that source familiarity doesn't lock in an unsuitable target.

After creating a shortlist, test it with representative data and access patterns. Review current [Azure service limits](/azure/azure-resource-manager/management/azure-subscription-service-limits), product limits, regional availability, and the [Azure pricing calculator](https://azure.microsoft.com/pricing/calculator/). Qualitative labels such as "low latency" or "lower management effort" aren't substitutes for workload measurements and a cost estimate.

### AWS migration starting points

The following mappings are starting points based on [Microsoft guidance for Azure and AWS database technologies](/azure/architecture/aws-professional/databases). They don't assert feature parity. Validate engine versions, extensions, data types, consistency behavior, operational processes, availability design, and migration tooling before choosing a target.

| AWS source or service | Azure candidates | Compatibility and migration focus |
| --- | --- | --- |
| Amazon RDS for SQL Server | Azure SQL Database, Azure SQL Managed Instance, or SQL Server on Azure Virtual Machines | Assess database-level versus instance-level compatibility and any need for operating-system control |
| Amazon RDS for PostgreSQL or Amazon Aurora PostgreSQL-Compatible Edition | Azure Database for PostgreSQL flexible server | Assess engine versions, extensions, SQL behavior, replication, and application dependencies |
| Amazon RDS for MySQL or Amazon Aurora MySQL-Compatible Edition | Azure Database for MySQL flexible server | Assess engine versions, SQL modes, features, replication, and application dependencies |
| Amazon DynamoDB | Azure Cosmos DB for NoSQL | Rework data modeling, partition keys, request patterns, consistency, and application APIs as needed |
| Amazon ElastiCache for Redis | Azure Managed Redis | Validate Redis feature use, clustering, persistence, networking, capacity, and migration sequencing |
| Amazon Neptune | Azure Cosmos DB for Apache Gremlin | Validate graph query compatibility, data model, limits, and client behavior |
| Amazon Redshift | Azure analytical services, based on the broader data platform design | Reassess warehouse, lake, ingestion, transformation, concurrency, and business intelligence requirements |
| Amazon Timestream | Azure Data Explorer or Microsoft Fabric Real-Time Intelligence | Reassess ingestion rate, retention, query patterns, streaming integration, and operational ownership |

Use the [Azure Database Migration Service tools matrix](dms/dms-tools-matrix.md) to identify assessment and migration tools for supported source-target combinations.

## Sample selection scenarios

These examples show how requirements produce a shortlist rather than a universal answer.

| Scenario | Important requirements | Initial shortlist | Reasoning and validation |
| --- | --- | --- | --- |
| New relational order-processing API | Multirow transactions, relationships, SQL queries, managed operations, and no instance-level dependency | Azure SQL Database; PostgreSQL or MySQL flexible server if an open-source engine is required | Select the engine ecosystem, then validate transaction behavior, availability, scale, and cost with a representative workload |
| Globally distributed customer profile store | Flexible JSON documents, predictable key-based access, geographic distribution, and explicit consistency choices | Azure Cosmos DB for NoSQL | Validate partition-key distribution, consistency, region layout, throughput, and cost; reconsider relational services if joins and broad relational transactions dominate |
| High-traffic product catalog | Durable relational or document system of record plus repeated low-latency reads | Primary database plus Azure Managed Redis | Keep authoritative writes in the primary database and validate cache invalidation, memory size, eviction, persistence, and failure behavior |
| Telemetry and time-series analysis | High-volume event ingestion, time-window aggregations, dashboards, and near-real-time exploration | Azure Data Explorer or Microsoft Fabric Real-Time Intelligence | Evaluate ingestion sources, query language, retention, Fabric integration, governance, and cost; use a separate OLTP service for business transactions |
| Amazon RDS for SQL Server migration | Existing SQL Server application, instance features, minimal code change, and managed operations | Azure SQL Managed Instance and Azure SQL Database | Run compatibility and feature assessments; choose Managed Instance only when required instance-level capabilities justify it |
| Amazon RDS for PostgreSQL migration | PostgreSQL compatibility, extensions, managed operations, and regional constraints | Azure Database for PostgreSQL flexible server | Validate versions, extensions, performance, availability, target region, and migration method before committing |

## Service fit and tradeoffs

### Azure SQL Database

[Azure SQL Database](/azure/azure-sql/database/sql-database-paas-overview) is a strong fit for managed, database-scoped relational applications using the SQL Server engine. It suits new cloud applications that value platform-managed availability, backups, patching, and scaling choices.

Don't select it solely because the source uses SQL Server. If the workload relies on instance-level features, compare [Azure SQL Database and Azure SQL Managed Instance](/azure/azure-sql/database/features-comparison). If it needs operating-system access, assess SQL Server on Azure Virtual Machines. Review current [Azure SQL Database resource limits](/azure/azure-sql/database/resource-limits-logical-server) and [pricing](https://azure.microsoft.com/pricing/details/azure-sql-database/) after choosing a deployment model.

### Azure SQL Managed Instance

[Azure SQL Managed Instance](/azure/azure-sql/managed-instance/sql-managed-instance-paas-overview) is a strong fit when a SQL Server migration needs managed operations and broader instance-level compatibility than Azure SQL Database provides.

Don't assume every SQL Server workload needs an instance boundary. For a database-scoped cloud application, Azure SQL Database might offer a simpler fit. For requirements that depend on full infrastructure control, include SQL Server on Azure Virtual Machines in the assessment. Validate compatibility and compare current pricing rather than treating Managed Instance as an automatic lift-and-shift target.

### Azure Database for PostgreSQL flexible server

[Azure Database for PostgreSQL flexible server](postgresql/overview.md) is a strong fit for managed relational workloads that require PostgreSQL compatibility, tooling, and extensions supported by the service. It provides managed operations while retaining controls that matter to PostgreSQL applications.

Don't use it when the application requires another engine's behavior, a nonrelational global partition model, or host-level control that the managed service doesn't expose. For specialized PostgreSQL-compatible requirements, you can also evaluate [Azure HorizonDB](horizondb/overview.md) where its preview status and capabilities meet your organization's policy. Compare [PostgreSQL hosting options](postgresql/configure-maintain/overview-postgres-choose-server-options.md) and current [pricing](https://azure.microsoft.com/pricing/details/postgresql/flexible-server/).

### Azure Database for MySQL flexible server

[Azure Database for MySQL flexible server](mysql/flexible-server/overview.md) is a strong fit for applications that require MySQL compatibility with managed backups, maintenance, availability, and scaling controls.

Don't use it when compatibility with another engine is required, when the workload needs a distributed nonrelational model, or when infrastructure-level control outweighs the benefits of a managed service. Validate versions, features, availability configuration, and current [pricing](https://azure.microsoft.com/pricing/details/mysql/flexible-server/) for the target region and workload.

### Azure Cosmos DB

[Azure Cosmos DB](/azure/cosmos-db/overview) is a strong fit for operational data that benefits from partitioning, geographic distribution, tunable consistency, and nonrelational models. Select an API from the model and compatibility requirements:

- **Azure Cosmos DB for NoSQL** is the native API for JSON document, key-value, and vector data models.
- **Azure Cosmos DB for MongoDB**, including request unit and vCore deployment options documented by the service, supports MongoDB-compatible application and migration scenarios.
- **Azure Cosmos DB for Apache Cassandra** supports Cassandra-compatible wide-column applications. If you need managed native open-source Cassandra clusters or hybrid Cassandra topology, also evaluate [Azure Managed Instance for Apache Cassandra](managed-instance-apache-cassandra/introduction.md).
- **Azure Cosmos DB for Apache Gremlin** supports graph data and Gremlin-compatible traversal scenarios.
- **Azure Cosmos DB for Table** supports key-value and Azure Table-compatible scenarios.

Compatibility APIs don't guarantee parity with every source-engine feature, version, client, or operational behavior. Don't use Azure Cosmos DB when relational joins, broad multirow relational transactions, or an unsuitable partitioning strategy dominate the workload. Review the current [API documentation](/azure/cosmos-db/), [service limits](/azure/cosmos-db/concepts-limits), and [pricing](https://azure.microsoft.com/pricing/details/cosmos-db/) for the selected API and capacity model.

### Azure Managed Redis

[Azure Managed Redis](/azure/redis/overview) is a strong fit for in-memory caching, session data, messaging, and related low-latency patterns. For cache-aside designs, pair it with a durable database and design explicit expiration, invalidation, and failure behavior.

Don't use Redis as the only system of record when the workload requires durable relational or document storage. Memory footprint, clustering, persistence, replicas, networking, and tier selection affect both performance and cost, so validate them against current [Azure Managed Redis documentation](/azure/redis/) and [pricing](https://azure.microsoft.com/pricing/details/managed-redis/).

For existing Azure Cache for Redis deployments, review the [retirement FAQ and migration guidance](/azure/azure-cache-for-redis/retirement-faq). Use Azure Managed Redis, not Azure Cache for Redis, as the candidate for new Redis designs.

### Analytical and time-series services

[Azure Data Explorer](/azure/data-explorer/data-explorer-overview) is a strong candidate for high-volume telemetry, logs, and time-series exploration. [Microsoft Fabric Real-Time Intelligence](/fabric/real-time-intelligence/overview) is a strong candidate when event-driven and real-time analytics should integrate with the Fabric platform.

Don't treat either analytical system as a drop-in replacement for a transactional database. Evaluate ingestion, query and retention patterns, integration, governance, operational ownership, and current pricing. An architecture can keep transactions in an OLTP service while moving events or replicated data into an analytical service.

## Related content

- [Prepare to choose a data store in Azure](/azure/architecture/guide/technology-choices/data-stores-getting-started)
- [Understand data models in Azure Architecture Center](/azure/architecture/data-guide/technology-choices/understand-data-store-models)
- [Azure Database Migration Service tools matrix](dms/dms-tools-matrix.md)
