---
title: PostgreSQL Extensions for Common Workloads
description: Compare common PostgreSQL extensions for vector, spatial, scheduling, auditing, and query-performance workloads on flexible server.
#customer intent: As a PostgreSQL user, I want to compare extensions by workload so that I can choose the right capability for my flexible server.
author: varun-dhawan
ms.author: varundhawan
ms.date: 07/24/2026
ms.service: azure-database-postgresql
ms.subservice: extensions
ms.topic: concept-article
---

# PostgreSQL extensions for common workloads overview

A PostgreSQL extension is a packaged capability that adds database features beyond the core engine. This guide helps you choose among five commonly used extensions for vector search, spatial data, job scheduling, audit logging, and query-performance investigation in Azure Database for PostgreSQL flexible server.

Azure Database for PostgreSQL flexible server supports a managed subset of PostgreSQL extensions. The five choices in this article are examples, not the complete catalog, and you can't bring arbitrary extensions to the service. Before you adopt an extension, verify its availability for your server's PostgreSQL version.

## Selection and enablement

Extension selection identifies the capability that fits your workload. Extension enablement makes that capability available on your server and in the databases that need it.

The managed-service enablement model has two levels. First, the server-level `azure.extensions` parameter must allow the extension. If the extension requires startup loading, its library must also be configured in `shared_preload_libraries`. Then, `CREATE EXTENSION` installs the extension objects separately in each target database. These requirements can affect operations, but they don't change which extension fits a workload.

## Workload comparison

The following table maps each workload to its primary extension choice. Check version support and the linked detailed guidance before implementation.

| Workload need | Extension | Primary fit | Decision-relevant caveat |
| --- | --- | --- | --- |
| Vector similarity search | pgvector (`vector`) | Stores vector data and supports similarity search with vector access methods. | pgvector is the common project name, but `vector` is the extension identifier used in `azure.extensions` and `CREATE EXTENSION`. |
| Geospatial processing | PostGIS | Adds geometry and geography spatial types and functions. | Confirm that PostGIS is available for your server's PostgreSQL version. |
| Scheduled database jobs | `pg_cron` | Schedules database maintenance and other SQL-based jobs. | Its library must be added to `shared_preload_libraries`, and the server must restart before the extension is created. |
| Security and compliance auditing | `pgaudit` | Produces detailed session and object audit logging. | Its library requires startup loading. Audit scope, log volume, destinations, and access are separate configuration decisions. |
| SQL execution statistics | `pg_stat_statements` | Tracks execution statistics to support query-performance investigation. | The service preloads the library, but you must still allow and create the extension. Statistics collection adds overhead and is distinct from Query Store and Query Performance Insight. |

For example, choose pgvector when an application compares embeddings by similarity, and use `vector` when you later allow and create the extension. Choose PostGIS instead when the application needs spatial types and functions for geometry or geography data.

For operational workloads, choose `pg_cron` for jobs that run on a database schedule, `pgaudit` when security or compliance requirements call for database audit records, and `pg_stat_statements` when you need PostgreSQL execution statistics for query investigation. Follow the extension-specific guidance to evaluate configuration and operational tradeoffs.

## Related content

- Extension availability and management: [Extensions and modules](concepts-extensions.md), [supported extensions by PostgreSQL version](concepts-extensions-by-engine.md), [supported extensions by name](concepts-extensions-versions.md), [allow extensions](how-to-allow-extensions.md), [load libraries](how-to-load-libraries.md), [create extensions](how-to-create-extensions.md), and [extension considerations](concepts-extensions-considerations.md)
- Vector search: [Enable and use pgvector](how-to-use-pgvector.md) and [optimize pgvector performance](how-to-optimize-performance-pgvector.md)
- Auditing and performance: [Audit logging](../security/security-audit.md), [PostgreSQL logs](../monitor/concepts-logging.md), [optimize query statistics collection](../troubleshoot/how-to-optimize-query-stats-collection.md), and [Query Performance Insight](../monitor/concepts-query-performance-insight.md)
