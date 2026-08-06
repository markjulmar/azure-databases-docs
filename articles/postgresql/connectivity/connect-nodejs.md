---
title: "Quickstart: Connect to PostgreSQL with Node.js"
description: Connect a Node.js application to Azure Database for PostgreSQL with the pg package, run a SELECT query, and verify the result.
#customer intent: As a Node.js developer, I want to connect to Azure Database for PostgreSQL so that I can query data from my application.
author: gkasar
ms.author: gkasar
ms.reviewer: maghan
ms.date: 08/06/2026
ms.service: azure-database-postgresql
ms.subservice: connectivity
ms.topic: quickstart
ai-usage: ai-generated
ms.devlang: javascript
---

# Quickstart: Connect and query Azure Database for PostgreSQL with Node.js

In this quickstart, you use the `pg` package to connect a Node.js application to an existing Azure Database for PostgreSQL flexible server over Transport Layer Security (TLS), run a `SELECT` query, and print the returned rows.

## Prerequisites

- An Azure account with an active subscription. [Create an Azure account for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- An Azure Database for PostgreSQL flexible server. If you need a server, [create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md).
- Network access from your development computer to the server. For public access, add your client IP address to the server firewall rules. For private access, run the application from a resource that can reach the server's virtual network. For more information, see [Networking for Azure Database for PostgreSQL](../network/how-to-networking.md).
- [Node.js](https://nodejs.org/en/download) and npm.

## Create the Node.js project

Create a Node.js project and install the `pg` package, which provides the PostgreSQL client used in this quickstart.

1. Open a terminal and create a project folder:

   ```bash
   mkdir postgresql-nodejs
   cd postgresql-nodejs
   ```

1. Install the `pg` package:

   ```bash
   npm install pg
   ```

## Get the PostgreSQL connection values

Get the host and sign-in values for your Azure Database for PostgreSQL flexible server from the Azure portal.

1. Sign in to the [Azure portal](https://portal.azure.com/).
1. Open your Azure Database for PostgreSQL flexible server.
1. On the server **Overview** page, copy the **Endpoint** and **Administrator login** values.
1. Use `postgres` for the database name unless you created another database. Use port `5432`.
1. Use the administrator password that you specified when you created the server. If you don't know the password, select **Reset password** on the server **Overview** page and set a new one.

Keep the password and any complete connection string out of your source files and source control.

## Set the connection environment variables

Set environment variables in the terminal where you'll run the Node.js application. Replace the example values with the connection values from the server **Overview** page.

### [Bash](#tab/bash)

1. Set the connection environment variables:

   ```bash
   export AZURE_POSTGRESQL_HOST="<endpoint>"
   export AZURE_POSTGRESQL_USER="<administrator-login>"
   export AZURE_POSTGRESQL_PASSWORD="<administrator-password>"
   export AZURE_POSTGRESQL_DATABASE="postgres"
   export AZURE_POSTGRESQL_PORT="5432"
   ```

1. Confirm that the host variable is set:

   ```bash
   echo "$AZURE_POSTGRESQL_HOST"
   ```

   The command should print the server endpoint. If it prints a blank line, set the variables again in the current terminal.

### [PowerShell](#tab/powershell)

1. Set the connection environment variables:

   ```powershell
   $env:AZURE_POSTGRESQL_HOST="<endpoint>"
   $env:AZURE_POSTGRESQL_USER="<administrator-login>"
   $env:AZURE_POSTGRESQL_PASSWORD="<administrator-password>"
   $env:AZURE_POSTGRESQL_DATABASE="postgres"
   $env:AZURE_POSTGRESQL_PORT="5432"
   ```

1. Confirm that the host variable is set:

   ```powershell
   $env:AZURE_POSTGRESQL_HOST
   ```

   The command should print the server endpoint. If it prints nothing, set the variables again in the current PowerShell session.

---

## Connect and run a SELECT query

Create and run an application that opens a TLS-protected connection, executes a verification query, prints the returned rows, and closes the client.

1. Create a file named *index.js* in the project folder and add the following code:

   ```javascript
   const { Client } = require("pg");

   const client = new Client({
     host: process.env.AZURE_POSTGRESQL_HOST,
     user: process.env.AZURE_POSTGRESQL_USER,
     password: process.env.AZURE_POSTGRESQL_PASSWORD,
     database: process.env.AZURE_POSTGRESQL_DATABASE,
     port: Number(process.env.AZURE_POSTGRESQL_PORT),
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
       console.error("Database query failed:", error);
       process.exitCode = 1;
     } finally {
       await client.end();
     }
   }

   queryDatabase().catch((error) => {
     console.error("Failed to close the database client:", error);
     process.exitCode = 1;
   });
   ```

   The `ssl` object requires the client to validate the server certificate. For certificate installation and advanced TLS configuration, see [Configure TLS connections](../security/security-tls-how-to-connect.md).

1. Run the application from the same terminal where you set the environment variables:

   ```bash
   node index.js
   ```

1. Confirm that the application prints a result similar to the following output:

   ```output
   [ { connection_test: 1 } ]
   ```

   If the connection fails, confirm that the environment variables contain the values from the server **Overview** page and that the server firewall or private network allows access from your computer. For a certificate error, follow the certificate guidance in [Configure TLS connections](../security/security-tls-how-to-connect.md), and then rerun the application.

## Clean up resources

This quickstart uses an existing flexible server and doesn't create Azure resources. When you no longer need the local project:

1. Close the terminal to clear the connection environment variables from the session.
1. Delete the *postgresql-nodejs* project folder.

## Related content

- [Connection libraries in Azure Database for PostgreSQL flexible server](concepts-connection-libraries.md)
- [Configure TLS connections in Azure Database for PostgreSQL flexible server](../security/security-tls-how-to-connect.md)
- [Quickstart: Connect and query with Python](connect-python.md)
- [Quickstart: Connect and query with Java](connect-java.md)
- [Quickstart: Connect and query with Go](connect-go.md)
