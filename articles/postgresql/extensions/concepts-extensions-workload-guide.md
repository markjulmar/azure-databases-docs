---
title: PostgreSQL Extensions for Common Workloads Overview
description: Choose PostgreSQL extensions for vector search, spatial data, scheduling, auditing, and query analysis in Azure Database for PostgreSQL.
#customer intent: As a developer or database administrator, I want to match a workload to a PostgreSQL extension so that I can choose the right capability.
author: varun-dhawan
ms.author: varundhawan
ms.date: 07/24/2026
ms.service: azure-database-postgresql
ms.subservice: extensions
ms.topic: concept-article
---

# PostgreSQL extensions for common workloads overview

This guide is a workload-based selection aid for PostgreSQL extensions in Azure Database for PostgreSQL flexible server. Use it to choose an extension before you follow the existing enablement and management procedures.

The extensions in this guide address common application and operational workloads. They aren't an exhaustive list of the extensions that flexible server supports.

## Workload selection

| Workload | Extension | Example scenario |
| --- | --- | --- |
| Vector similarity search | `vector` (commonly called pgvector) | An application stores embeddings and finds items that are semantically similar to a query. |
| Spatial data and functions | `postgis` | An application stores geometry or geography data and runs spatial queries. |
| Scheduled database jobs | `pg_cron` | A database administrator schedules recurring maintenance or data-management jobs inside PostgreSQL. |
| Database audit logging | `pgaudit` | A security administrator records session or object activity for security and compliance review. |
| Query workload visibility | `pg_stat_statements` | A performance engineer examines SQL statement execution statistics to understand a query workload. |

`vector` and `postgis` add data types and functions that applications use directly. `pg_cron`, `pgaudit`, and `pg_stat_statements` support database operations, security, and performance analysis. If your scenario combines categories, you might use more than one extension.

For example, choose `vector` when an application needs nearest-neighbor or semantic similarity search. The PostgreSQL community commonly calls the project pgvector, but `vector` is the name used in the server allowlist and by `CREATE EXTENSION`. For implementation guidance, see [Enable and use pgvector](how-to-use-pgvector.md), [Optimize performance when using pgvector](how-to-optimize-performance-pgvector.md), and [Create a semantic search](../azure-ai/generative-ai-semantic-search.md).

For an operational example, choose `pgaudit` when you need session or object audit records rather than general server diagnostics. See [Audit logging](../security/security-audit.md) for audit configuration and [server logging](../monitor/concepts-logging.md) for log destinations. Choose `pg_stat_statements` when you need execution statistics for statements in the current statistics view. If you need persisted query history and runtime statistics, compare it with [Query Store](../monitor/concepts-query-store.md).

## Extension support and activation

Extension availability and versions depend on the PostgreSQL version of your server. Check the [supported extensions by name](concepts-extensions-versions.md) or [supported extensions by PostgreSQL version](concepts-extensions-by-engine.md) before you choose an extension. These references are the source of truth for support rather than the examples in this guide.

Extension activation has two levels:

- At the server level, the `azure.extensions` parameter is an allowlist gate. An extension must be on this allowlist before extension management commands can run. See [Allow extensions](how-to-allow-extensions.md).
- At the database level, `CREATE EXTENSION` installs the extension objects in the connected database. Install the extension separately in each database that needs it. See [Create extensions](how-to-create-extensions.md).

Some choices have an additional server-readiness requirement. `pg_cron` and `pgaudit` must be included in `shared_preload_libraries`, and changing this static parameter requires a server restart. `pg_stat_statements` is already preloaded on flexible server, but you must still allowlist it and create it in each target database. See [Load libraries](how-to-load-libraries.md) for the procedure and [extension considerations](concepts-extensions-considerations.md) for extension-specific requirements.

## Related content

- [Extensions and modules in Azure Database for PostgreSQL flexible server](concepts-extensions.md)
- [Supported extensions by name](concepts-extensions-versions.md)
- [Considerations when using extensions and modules](concepts-extensions-considerations.md)
