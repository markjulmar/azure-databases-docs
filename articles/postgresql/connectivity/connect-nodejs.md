---
title: Connect Node.js to Azure Database for PostgreSQL
description: Connect a Node.js application to Azure Database for PostgreSQL flexible server with pg, explicit TLS, and a printed SELECT result.
#customer intent: As a Node.js developer new to Azure, I want to connect to an Azure Database for PostgreSQL flexible server so that I can query data from my application.
author: gkasar
ms.author: gkasar
ms.reviewer: maghan
ms.date: 09/08/2026
ms.service: azure-database-postgresql
ms.subservice: connectivity
ms.topic: quickstart
ai-usage: ai-generated
ms.custom:
  - mvc
  - mode-api
  - devx-track-js
ms.devlang: "javascript"
---

# Quickstart: Connect Node.js to Azure Database for PostgreSQL flexible server

Azure Database for PostgreSQL flexible server is a managed service for running, managing, and scaling PostgreSQL databases in the cloud. A working application connection lets you start querying your database from Node.js.

In this quickstart, you install the `pg` package, connect a Node.js application to an existing flexible server with Transport Layer Security (TLS), run a `SELECT` statement, and print its result to verify the connection. This quickstart uses PostgreSQL password authentication and a direct connection on port `5432`.

## Prerequisites

- An Azure account with an active subscription. If you don't have one, [create an Azure account for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- An Azure Database for PostgreSQL flexible server. If you need a server, [create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md).
- Network access from your development environment to the server. For public access, add your current client IP address to the server firewall rules. For private access, connect from a resource in the same virtual network. For setup guidance, see [Networking for Azure Database for PostgreSQL](../network/how-to-networking.md).
- Node.js and npm installed on your development environment. The `pg` project supports Node.js versions that are in long-term support (LTS).
<!-- TODO: [C3] [C4] SME: confirm the exact currently supported Node.js version or LTS range for the pg release used by this quickstart. -->
- The administrator user name and password for the flexible server.

## Install pg

Create a local project and install the `pg` package. The package provides the PostgreSQL client used throughout this quickstart.

1. Create a folder for the quickstart, and change to that folder.
1. Install `pg`:

   ```bash
   npm install pg
   ```

1. Confirm that npm completes without an installation error.

For information about other supported client libraries, see [Connection libraries for Azure Database for PostgreSQL](concepts-connection-libraries.md).

## Get the connection information

Collect the values that the Node.js application needs before you create the connection.

1. In the [Azure portal](https://portal.azure.com), open your Azure Database for PostgreSQL flexible server.
1. On the server **Overview** page, copy the **Endpoint** and **Administrator login** values.
1. Choose the database to query. Use `postgres` if you want to connect to the default database created with the server.
1. Retain the server password securely. Don't add the password to the JavaScript file or print it in command output.

Use these values for the standard `pg` environment variables:

| Environment variable | Value |
| --- | --- |
| `PGHOST` | The server **Endpoint** value. |
| `PGPORT` | `5432` for a direct server connection. |
| `PGDATABASE` | The database name, such as `postgres`. |
| `PGUSER` | The **Administrator login** value. |
| `PGPASSWORD` | The server password. |

## Create the Node.js application

Create an application that reads its connection information from environment variables, requests TLS explicitly, runs one query, and closes the client.

> [!IMPORTANT]
> The sample explicitly requests an encrypted TLS connection. Before you use this pattern in production, configure a trusted certificate authority (CA) and hostname verification. Encryption without verified server identity doesn't protect against server spoofing.

1. Create a file named `connect.mjs` in the project folder.
1. Add the following code to `connect.mjs`:

   ```javascript
   import pg from 'pg';

   const { Client } = pg;

   const client = new Client({
     host: process.env.PGHOST,
     port: Number(process.env.PGPORT ?? 5432),
     database: process.env.PGDATABASE,
     user: process.env.PGUSER,
     password: process.env.PGPASSWORD,
     ssl: true,
   });

   let connected = false;

   try {
     await client.connect();
     connected = true;

     const result = await client.query(
       "SELECT 'Connected to Azure Database for PostgreSQL' AS message"
     );

     console.log(result.rows[0].message);
   } catch (error) {
     console.error('Connection failed:', error.message);
     process.exitCode = 1;
   } finally {
     if (connected) {
       await client.end();
     }
   }
   ```

<!-- TODO: [QS1] [QS2] SME: supply or approve the exact pg TLS configuration that loads the appropriate Azure root CA, verifies the server hostname, and resolves the interaction between an ssl object and sslmode or related connection-string parameters. -->
<!-- TODO: Evidence conflict [QS2]: the Azure TLS article describes generic PostgreSQL client behavior, the pg upgrade guide describes ssl:true behavior beginning with pg 8, and libpq distinguishes encryption from verify-ca and verify-full. These sources don't establish one exact CA-loading configuration for this pg sample. -->
<!-- TODO: SME review required [QS1] [QS2] [QS3]: run the assembled ESM sample against Azure Database for PostgreSQL flexible server and confirm the import form, TLS behavior, client closure path, and expected output with the current pg release. -->

## Run the application

Supply the connection values as environment variables when you run the Node.js application. This approach keeps the password out of the source file and printed output.

1. Set `PGHOST`, `PGPORT`, `PGDATABASE`, `PGUSER`, and `PGPASSWORD` in your current shell. Use the values you collected from the server **Overview** page, set `PGPORT` to `5432`, and set `PGDATABASE` to your database name.
1. Run the application from the project folder:

   ```bash
   node connect.mjs
   ```

1. Confirm that the application prints the literal value selected by the query:

   ```output
   Connected to Azure Database for PostgreSQL
   ```

If the application prints `Connection failed`, confirm that the environment-variable values are correct and that your public firewall rule or private network path allows the development environment to reach the server. Then run the application again.

## Clean up resources

Remove resources that you don't want to retain after you finish the quickstart.

1. Close the terminal session to clear connection values that you set only for that session.
1. Delete the local project folder to remove `connect.mjs`, the installed packages, and npm-generated project files.
1. If you created an Azure Database for PostgreSQL flexible server for this quickstart, choose the cleanup scope:
   - Delete only the flexible server to retain other resources in its resource group.
   - Delete the resource group to remove the server and every other resource in that group.

For the Azure cleanup commands, see [Clean up resources in the create-server quickstart](../configure-maintain/quickstart-create-server.md#clean-up-resources).

## Next step

> [!div class="nextstepaction"]
> [Connect and query Azure Database for PostgreSQL](how-to-connect-query-guide.md)

## Related content

- [Configure TLS for Azure Database for PostgreSQL](../security/security-tls-how-to-connect.md)
- [Connection libraries for Azure Database for PostgreSQL](concepts-connection-libraries.md)
- [Connection pooling best practices](concepts-connection-pooling-best-practices.md)

For applications that run frequent queries, `pg` includes application-level pooling through `pg-pool`. Azure PgBouncer is a separate server-side pooling option. Review [PgBouncer in Azure Database for PostgreSQL](concepts-pgbouncer.md) after the direct connection works.
