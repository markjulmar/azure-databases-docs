---
title: "Quickstart: Connect to PostgreSQL with Node.js and pg"
description: Connect a Node.js application to Azure Database for PostgreSQL with pg, TLS certificate validation, and a simple SELECT query.
#customer intent: As a Node.js developer, I want to connect to Azure Database for PostgreSQL, so that I can query my database from an application.
author: gkasar
ms.author: gkasar
ms.reviewer: maghan
ms.date: 08/06/2026
ms.service: azure-database-postgresql
ms.subservice: connectivity
ms.topic: quickstart
ms.devlang: javascript
ai-usage: ai-generated
---

# Quickstart: Connect and query Azure Database for PostgreSQL with Node.js

In this quickstart, you create a minimal Node.js project, configure a TLS-protected connection, run a `SELECT` query, print the result, and close the database client by using the `pg` package.

Azure Database for PostgreSQL flexible server is a managed PostgreSQL service that Node.js applications can access by using the `pg` package.

## Prerequisites

To complete this quickstart, you need:

- An Azure account with an active subscription. [Create an Azure account for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- An [Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md) and a database, such as the default *postgres* database.
- A PostgreSQL administrator login or another database user, and the corresponding password.
- Network access from your development computer to the server. For public access, add your client IP address to a server firewall rule. For private access, use a computer that can reach the server through the configured private network.
- A supported version of [Node.js and npm](https://nodejs.org/en/download).

> [!IMPORTANT]
> A server configured with **Public access (allowed IP addresses)** is the simplest option for this quickstart. **Private access (VNet Integration)** requires a suitable private network path, which this quickstart doesn't configure. For either option, see [Networking in Azure Database for PostgreSQL flexible server](../network/how-to-networking.md).

## Create a Node.js project and install pg

Create a minimal project and install `pg` as the project's only PostgreSQL client.

1. Open a terminal and create a project directory:

   ```bash
   mkdir postgres-nodejs
   cd postgres-nodejs
   ```

1. Create the project's *package.json* file:

   ```bash
   npm init -y
   ```

1. Install the `pg` package:

   ```bash
   npm install pg
   ```

   When the installation succeeds, *package.json* lists `pg` under `dependencies`.

## Get PostgreSQL connection information

Get the server endpoint, user name, database name, and password before you configure the Node.js application.

1. Sign in to the [Azure portal](https://portal.azure.com/).
1. Search for and open your Azure Database for PostgreSQL flexible server.
1. On the server's **Overview** page, copy **Endpoint** and **Administrator login**.
1. Record the database name. Use *postgres* if you haven't created another database.
1. Keep the password for the administrator login or database user available. If you forgot the administrator password, select **Reset password** on the server's **Overview** page.
1. Review the connection string information in the portal to confirm the endpoint, port `5432`, database name, and user name. The sample uses these values separately instead of copying a connection string.

## Set environment variables

Store the connection values in environment variables so that credentials don't appear in source code. Replace each placeholder with the value you recorded from the Azure portal.

### [Windows](#tab/windows)

1. In PowerShell, set the connection environment variables:

   ```powershell
   $env:PGHOST = "<server-endpoint>"
   $env:PGPORT = "5432"
   $env:PGDATABASE = "<database-name>"
   $env:PGUSER = "<database-user>"
   $env:PGPASSWORD = "<database-password>"
   ```

1. Confirm that the nonsecret values are set:

   ```powershell
   $env:PGHOST
   $env:PGDATABASE
   $env:PGUSER
   ```

### [macOS/Linux](#tab/macos-linux)

1. In Bash, set the connection environment variables:

   ```bash
   export PGHOST="<server-endpoint>"
   export PGPORT="5432"
   export PGDATABASE="<database-name>"
   export PGUSER="<database-user>"
   export PGPASSWORD="<database-password>"
   ```

1. Confirm that the nonsecret values are set:

   ```bash
   printf '%s\n' "$PGHOST" "$PGDATABASE" "$PGUSER"
   ```

---

These variables apply only to the current terminal session. Don't print or commit `PGPASSWORD`.

## Configure TLS and query PostgreSQL

Use a `pg` TLS configuration object that requires certificate validation. Azure Database for PostgreSQL flexible server enforces encrypted connections and supports TLS 1.2 and TLS 1.3.

1. In the *postgres-nodejs* project directory, create a file named *index.js* with the following code:

   ```javascript
   const { Client } = require("pg");

   const client = new Client({
     host: process.env.PGHOST,
     port: Number(process.env.PGPORT),
     database: process.env.PGDATABASE,
     user: process.env.PGUSER,
     password: process.env.PGPASSWORD,
     ssl: {
       rejectUnauthorized: true,
     },
   });

   async function queryDatabase() {
     try {
       await client.connect();
       const result = await client.query("SELECT 1 AS connection_test;");
       console.log(result.rows);
     } catch (error) {
       console.error("Database connection or query failed:", error.message);
       process.exitCode = 1;
     } finally {
       await client.end();
     }
   }

   queryDatabase();
   ```

   The sample sets `ssl` directly and doesn't use `sslmode` in a connection string. Don't combine a `pg` TLS configuration object with `sslmode`, `sslcert`, `sslkey`, or `sslrootcert` connection-string parameters because those parameters can replace the TLS object configuration.

1. If your Node.js environment doesn't trust the root certificate authorities used by the server, follow [Configure TLS connection in Azure Database for PostgreSQL flexible server](../security/security-tls-how-to-connect.md) to install the required root certificates. Keep certificate verification enabled.

## Run and verify the Node.js application

Run the application from the terminal where you set the connection environment variables.

1. From the *postgres-nodejs* project directory, run:

   ```bash
   node index.js
   ```

1. Confirm that the application prints the query result:

   ```output
   [ { connection_test: 1 } ]
   ```

   If the connection fails, use the error message to check the endpoint, credentials, firewall or private network path, and trusted root certificates. After you correct the configuration, run `node index.js` again.

The `finally` block closes the `pg` client whether the connection and query succeed or fail. For applications that handle concurrent requests, the `pg` package also provides `Pool`; see [Connection pooling best practices](concepts-connection-pooling-best-practices.md) before you design a production connection strategy.

## Clean up resources

The sample closes its database client automatically. Delete the local *postgres-nodejs* project directory when you no longer need it.

If you created a resource group only for the linked server-creation quickstart, you can delete that resource group and all its resources. Replace `<resource-group-name>` with its name:

```azurecli-interactive
az group delete \
    --name "<resource-group-name>" \
    --yes
```

Don't delete a resource group that contains resources you want to keep.

## Related content

- [Connection libraries for Azure Database for PostgreSQL](concepts-connection-libraries.md)
- [Configure TLS connection in Azure Database for PostgreSQL flexible server](../security/security-tls-how-to-connect.md)
- [Quickstart: Use Python to connect and query data in Azure Database for PostgreSQL](connect-python.md)
- [Quickstart: Use Java to connect and query data in Azure Database for PostgreSQL](connect-java.md)
- [Quickstart: Use .NET to connect and query data in Azure Database for PostgreSQL](connect-csharp.md)
- [Quickstart: Use Go to connect and query data in Azure Database for PostgreSQL](connect-go.md)
- [Quickstart: Use PHP to connect and query data in Azure Database for PostgreSQL](connect-php.md)
