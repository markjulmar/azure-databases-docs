---
title: "Connect and Query a Flexible Server With Node.js"
description: Connect a Node.js application to Azure Database for PostgreSQL flexible server with pg, TLS certificate validation, and a SELECT query.
#customer intent: As a Node.js developer, I want to connect to Azure Database for PostgreSQL flexible server so that I can query my database from an application.
author: gkasar
ms.author: gkasar
ms.reviewer: maghan
ms.date: 08/10/2026
ms.service: azure-database-postgresql
ms.subservice: connectivity
ms.topic: quickstart
ai-usage: ai-generated
ms.devlang: "javascript"
---

# Quickstart: Connect and query Azure Database for PostgreSQL flexible server with Node.js

In this quickstart, you connect a Node.js application to Azure Database for PostgreSQL flexible server, run a `SELECT` query, and print the result. The application uses the `pg` package and validates the server's Transport Layer Security (TLS) certificate.

## Prerequisites

- An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- An existing [Azure Database for PostgreSQL flexible server and database](../configure-maintain/quickstart-create-server.md).
- Network access from your client to the server. For public access, add your client IP address to the server's firewall rules. For private access, run the client from a resource that can reach the server through its virtual network. For more information, see [Networking in Azure Database for PostgreSQL flexible server](../network/how-to-networking.md).
- A supported version of [Node.js](https://nodejs.org/) with npm.

This quickstart assumes that you're familiar with Node.js application development.

## Set up the Node.js project

Create a Node.js project and install `pg`, the PostgreSQL client package used in this quickstart.

1. Open a terminal and create a project directory.

   ```bash
   mkdir postgres-nodejs-quickstart
   cd postgres-nodejs-quickstart
   ```

1. Initialize the project if the directory doesn't already contain a *package.json* file.

   ```bash
   npm init -y
   ```

1. Install the `pg` package.

   ```bash
   npm install pg
   ```

1. Confirm that npm lists the installed package before you continue.

   ```bash
   npm list pg
   ```

   If npm reports that `pg` is missing, run `npm install pg` again from the directory that contains *package.json*.

## Get the PostgreSQL connection values

Get the server endpoint and administrator login from your Azure Database for PostgreSQL flexible server in the Azure portal.

1. In the [Azure portal](https://portal.azure.com/), select **All resources**, and then select your Azure Database for PostgreSQL flexible server.
1. On the server's **Overview** page, copy the **Endpoint** value. The endpoint resembles `my-server.postgres.database.azure.com`.
1. Copy the **Administrator login** value.
1. Note the database name. You can use the default `postgres` database or an existing database on the server.
1. Have the password for the administrator login ready. The portal doesn't show the current password. If you forgot it, use **Reset password** on the server's **Overview** page.

Keep these values available for the environment variables in the next section.

## Configure the connection and TLS certificate

Store the connection values outside your source code and configure `pg` to validate the server certificate.

1. Follow [Configure TLS connections in Azure Database for PostgreSQL flexible server](../security/security-tls-how-to-connect.md#download-and-convert-root-ca-certificates) to obtain the required root certificate authority (CA) certificates and save the combined certificates in a Privacy Enhanced Mail (PEM) file.

1. Set the connection environment variables. Replace each value with the endpoint, login, database name, password, and PEM file path for your environment.

   ### [Windows PowerShell](#tab/powershell)

   ```powershell
   $env:PGHOST="<server-endpoint>"
   $env:PGPORT="5432"
   $env:PGDATABASE="<database-name>"
   $env:PGUSER="<administrator-login>"
   $env:PGPASSWORD="<password>"
   $env:PGSSLROOTCERT="<path-to-root-ca-pem-file>"
   ```

   ### [macOS/Linux](#tab/bash)

   ```bash
   export PGHOST="<server-endpoint>"
   export PGPORT="5432"
   export PGDATABASE="<database-name>"
   export PGUSER="<administrator-login>"
   export PGPASSWORD="<password>"
   export PGSSLROOTCERT="<path-to-root-ca-pem-file>"
   ```

   ---

   Don't add the password or connection values to source control. If a path contains spaces, keep the quotation marks around the value.

1. Confirm that all six environment variables contain values in the terminal where you'll run the application.

   ### [Windows PowerShell](#tab/powershell)

   ```powershell
   "PGHOST", "PGPORT", "PGDATABASE", "PGUSER", "PGPASSWORD", "PGSSLROOTCERT" |
     ForEach-Object {
       if (Test-Path "Env:$_") { "$_ is set" } else { "$_ is not set" }
     }
   ```

   ### [macOS/Linux](#tab/bash)

   ```bash
   for name in PGHOST PGPORT PGDATABASE PGUSER PGPASSWORD PGSSLROOTCERT; do
     test -n "$(printenv "$name")" && echo "$name is set" || echo "$name is not set"
   done
   ```

   ---

   If a value is empty, set that environment variable before you continue.

## Connect and run a SELECT query

Create an application that opens a TLS-secured connection, queries the current database and user, prints the returned row, and closes the connection.

1. Create a file named *index.js* in the project directory, and add the following code:

   ```javascript
   const fs = require("node:fs");
   const { Client } = require("pg");

   const client = new Client({
     host: process.env.PGHOST,
     port: Number(process.env.PGPORT),
     database: process.env.PGDATABASE,
     user: process.env.PGUSER,
     password: process.env.PGPASSWORD,
     ssl: {
       ca: fs.readFileSync(process.env.PGSSLROOTCERT, "utf8"),
       rejectUnauthorized: true,
     },
   });

   async function main() {
     try {
       await client.connect();
       console.log("Connection established with TLS.");

       const result = await client.query(
         "SELECT current_database() AS database_name, current_user AS user_name;"
       );
       console.log(result.rows[0]);
     } finally {
       await client.end();
     }
   }

   main().catch((error) => {
     console.error("Connection or query failed:", error.message);
     process.exitCode = 1;
   });
   ```

1. Run the application from the directory that contains *index.js*.

   ```bash
   node index.js
   ```

1. Confirm that the output starts with the connection message and includes one row with your database and user values:

   ```output
   Connection established with TLS.
   { database_name: '<database-name>', user_name: '<user-name>' }
   ```

   If the connection fails, check that the environment variables are set in the same terminal, the PEM file path is valid, and the server's firewall or virtual network permits access from your client.

## Clean up resources

This quickstart uses an existing Azure Database for PostgreSQL flexible server and doesn't create Azure resources. Keep the server and project if you plan to continue using them.

To remove the local secrets after you finish, close the terminal. You can also delete the *postgres-nodejs-quickstart* project directory if you no longer need it. If you created the server only for this quickstart, see [Delete an Azure Database for PostgreSQL flexible server](../configure-maintain/how-to-delete-server.md).

## Related content

- [Configure TLS connections in Azure Database for PostgreSQL flexible server](../security/security-tls-how-to-connect.md)
- [Connection pooling best practices](concepts-connection-pooling-best-practices.md)
- [Quickstart: Use Python to connect and query data in Azure Database for PostgreSQL](connect-python.md)
- [Quickstart: Use Java and JDBC in Azure Database for PostgreSQL flexible server](connect-java.md)
- [Quickstart: Use .NET to connect and query data in Azure Database for PostgreSQL](connect-csharp.md)
- [Quickstart: Use Go to connect and query data in Azure Database for PostgreSQL](connect-go.md)
- [Quickstart: Use PHP to connect and query data in Azure Database for PostgreSQL](connect-php.md)
- [Quickstart: Connect and query by using Azure CLI](connect-azure-cli.md)
