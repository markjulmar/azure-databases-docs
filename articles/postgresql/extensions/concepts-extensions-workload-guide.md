---
title: PostgreSQL Extensions for Common Workloads
description: Compare PostgreSQL extensions for vector search, geospatial data, scheduled jobs, auditing, and query performance analysis.
#customer intent: As a developer or database administrator, I want to compare PostgreSQL extensions so that I can select one for my workload.
author: akashraokm
ms.author: akashrao
ms.date: 08/21/2026
ms.service: azure-database-postgresql
ms.subservice: extensions
ms.topic: concept-article
ai-usage: ai-generated
---

# PostgreSQL extensions for common workloads overview

A PostgreSQL extension workload guide maps a database requirement to an extension that adds the required capability. For Azure Database for PostgreSQL flexible server, this guide compares extensions for vector search, geospatial data, scheduled jobs, audit logging, and query performance analysis.

Use the comparison and examples to choose an extension and understand its server-level and database-level enablement boundaries. Follow the linked articles for setup, configuration, support, and workload procedures.

## Extension selection by workload

The following table maps each workload to the extension that directly supports it.

| Workload | Extension | Capability |
| --- | --- | --- |
| Vector similarity and nearest-neighbor search | pgvector (`vector`) | Adds a vector data type and the IVFFlat and HNSW access methods. |
| Geospatial data | PostGIS (`postgis`) | Adds geometry and geography spatial types and functions. |
| Scheduled database jobs | `pg_cron` | Schedules jobs that run inside PostgreSQL. |
| Database audit logging | `pgaudit` | Provides detailed session and object audit logging. |
| SQL execution-statistics analysis | `pg_stat_statements` | Tracks SQL statement execution statistics for workload analysis. |

## Server and database enablement boundaries

PostgreSQL extension enablement in Azure Database for PostgreSQL flexible server has separate server and database boundaries. At the server boundary, the `azure.extensions` allowlist controls whether extension data definition language (DDL) commands are permitted. At the database boundary, `CREATE EXTENSION` deploys an extension's SQL objects only in the database where the command runs. Allowlisting an extension doesn't make its SQL objects available in every database.

Some extensions also require a shared library at server startup. Among the choices in this guide, `pg_cron` and `pgaudit` require configuration through `shared_preload_libraries`. The `pg_stat_statements` library is preloaded on every flexible server, but the extension still requires allowlisting and creation in each database where its objects are needed. For detailed enablement and support information, see [Allow extensions](how-to-allow-extensions.md), [Create extensions](how-to-create-extensions.md), [Load libraries](how-to-load-libraries.md), and [Troubleshoot extension errors](errors-extensions.md).

## Workload selection examples

The following examples distinguish the workload each extension addresses and the main factor to consider before selecting it.

### Vector similarity search with pgvector

Choose pgvector when an application stores embeddings and needs vector similarity or nearest-neighbor search, such as semantic search or retrieval-augmented generation. The community project is called pgvector, but its extension identifier is `vector`; use `vector` when allowlisting or creating the extension. For implementation and tuning guidance, see [Enable and use pgvector](how-to-use-pgvector.md) and [Optimize pgvector performance](how-to-optimize-performance-pgvector.md).

### Geospatial data with PostGIS

Choose PostGIS when tables need geometry or geography spatial types and functions. This capability fits workloads that store and query spatial data rather than workloads centered on scheduling, auditing, or SQL execution statistics. Check the [supported extensions by PostgreSQL version](concepts-extensions-by-engine.md) before adopting the extension.

### Scheduled database jobs with pg_cron

Choose `pg_cron` when PostgreSQL should run scheduled database work, such as periodic vacuum operations or old-data cleanup. `pg_cron` runs at most one instance of the same job at a time and queues an overlapping run until the current run finishes. It also requires startup library loading and a server restart before extension creation. See [pg_cron considerations](concepts-extensions-considerations.md#pg_cron) for configuration and usage details.

### Audit logging with pgaudit

Choose `pgaudit` when security or compliance requirements call for detailed session or object audit logging. Its output consists of audit entries in PostgreSQL logs, so the workload also needs an appropriate log destination and query approach. The extension requires startup library loading in addition to allowlisting and database-level creation. See [Audit logging](../security/security-audit.md) for configuration and log access guidance.

### Query performance analysis with pg_stat_statements

Choose `pg_stat_statements` when monitoring or troubleshooting requires SQL execution statistics, such as identifying queries that consume high input/output resources. The extension observes query execution and adds runtime overhead. Query Store provides a related mechanism, and using only one of the two avoids the overhead of collecting the same type of information through both. See [Optimize query statistics collection](../troubleshoot/how-to-optimize-query-stats-collection.md) for configuration guidance.

## Related content

- [Extensions and modules](concepts-extensions.md)
- [Extension and module considerations](concepts-extensions-considerations.md)
- [Extensions and modules by name](concepts-extensions-versions.md)
