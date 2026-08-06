---
title: Connect Node.js to PostgreSQL with the pg Client
description: Connect a Node.js application to Azure Database for PostgreSQL with the pg client, TLS, a test query, and secure credential handling.
#customer intent: As a Node.js developer, I want to connect to Azure Database for PostgreSQL flexible server so that I can verify database access from my application.
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

# Quickstart: Query data in Azure Database for PostgreSQL with Node.js

In this quickstart, you connect a Node.js application to Azure Database for PostgreSQL flexible server by using the `pg` client, run a `SELECT 1` query, and verify the result.

This quickstart assumes that you're familiar with Node.js development but are new to Azure Database for PostgreSQL.

## Prerequisites

- An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- An existing [Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md).
- Network access from your client to the flexible server:
  - For **Public access (allowed IP addresses)**, add your client IP address to the server's firewall rules.
  - For **Private access (VNET Integration)**, run the sample from a resource that can reach the server's virtual network.

  For configuration details, see [Networking in Azure Database for PostgreSQL flexible server](../network/how-to-networking.md).
- A supported Node.js release with npm installed.
- The administrator password or another PostgreSQL user credential that can connect to the database.

## Set up the Node.js project

Create a minimal Node.js project and install the `pg` package.

1. Create a project directory, and then change to that directory:

   ```bash
   mkdir node-postgresql-quickstart
   cd node-postgresql-quickstart
   ```

1. Install the `pg` package:

   ```bash
   npm install pg
   ```

## Get connection information from the Azure portal

The Node.js sample needs the server endpoint, administrator login, database name, and password.

1. In the [Azure portal](https://portal.azure.com/), search for and select your Azure Database for PostgreSQL flexible server.
1. On the server's **Overview** page, copy the values shown as **Endpoint** and **Administrator login**.
1. Choose the database to connect to. You can use the default *postgres* database or another database that you created.
1. Locate the password for the administrator login. If you forgot it, use **Reset password** on the **Overview** page.

Keep these values available for the next section. Don't add the password to source code or commit it to source control.

## Configure the connection values

Set the connection values as environment variables in the terminal where you'll run the sample. Replace each bracketed value with the value you obtained in the preceding section.

### PowerShell

1. Set the endpoint, port, database name, and administrator login:

   ```powershell
   $env:PGHOST = "<endpoint>"
   $env:PGPORT = "5432"
   $env:PGDATABASE = "<database-name>"
   $env:PGUSER = "<administrator-login>"
   ```

1. Set the password environment variable:

   ```powershell
   $env:PGPASSWORD = Read-Host "Enter the PostgreSQL password" -MaskInput
   ```

### Bash

1. Set the endpoint, port, database name, and administrator login:

   ```bash
   export PGHOST="<endpoint>"
   export PGPORT="5432"
   export PGDATABASE="<database-name>"
   export PGUSER="<administrator-login>"
   ```

1. Set the password environment variable:

   ```bash
   read -s -p "Enter the PostgreSQL password: " PGPASSWORD
   echo
   export PGPASSWORD
   ```

The environment variables apply to the current terminal session. The sample reads them at run time and explicitly sets `ssl: true`. Connections to Azure Database for PostgreSQL flexible server use Transport Layer Security (TLS), and the server enforces secure transport by default. For certificate and root certificate configuration, see [Configure TLS connections in Azure Database for PostgreSQL flexible server](../security/security-tls-how-to-connect.md).

## Connect and run a query

Create the application, connect with TLS, run `SELECT 1`, print the result, and close the client.

1. In the *node-postgresql-quickstart* directory, create *index.js* with the following code:

   ```javascript
   const { Client } = require("pg");

   async function main() {
     const requiredVariables = [
       "PGHOST",
       "PGDATABASE",
       "PGUSER",
       "PGPASSWORD",
     ];

     for (const variable of requiredVariables) {
       if (!process.env[variable]) {
         throw new Error(`Set the ${variable} environment variable before running the sample.`);
       }
     }

     const client = new Client({
       host: process.env.PGHOST,
       port: Number(process.env.PGPORT ?? "5432"),
       user: process.env.PGUSER,
       password: process.env.PGPASSWORD,
       database: process.env.PGDATABASE,
       ssl: true,
     });

     try {
       await client.connect();
       const result = await client.query("SELECT 1 AS connection_test;");
       console.log(result.rows);
     } catch (error) {
       console.error("Connection or query failed:", error);
       process.exitCode = 1;
     } finally {
       await client.end();
     }
   }

   main().catch((error) => {
     console.error("Application failed:", error);
     process.exitCode = 1;
   });
   ```

1. Run the application:

   ```bash
   node index.js
   ```

1. Confirm that a successful run prints the query result:

   ```output
   [ { connection_test: 1 } ]
   ```

If the application reports a connection error, confirm that the environment variable values are correct and that the client network has access to the flexible server. Don't continue until the query returns `connection_test: 1`.

## Clean up resources

If you created the flexible server only for this quickstart, delete the server or its resource group to avoid charges. For instructions to delete only the server, see [Delete a server in Azure Database for PostgreSQL flexible server](../configure-maintain/how-to-delete-server.md). If you used an existing server, keep the server and remove the local *node-postgresql-quickstart* directory when you no longer need it. Close the terminal session to clear the connection environment variables.

## Related content

- [Connection libraries in Azure Database for PostgreSQL flexible server](concepts-connection-libraries.md)
- [Configure TLS connections in Azure Database for PostgreSQL flexible server](../security/security-tls-how-to-connect.md)
- [Connection pooling best practices for Azure Database for PostgreSQL flexible server](concepts-connection-pooling-best-practices.md)
