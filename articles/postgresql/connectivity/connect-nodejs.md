---
title: "Quickstart: Connect and query Azure Database for PostgreSQL with Node.js"
description: Connect a Node.js application to Azure Database for PostgreSQL flexible server over TLS, run a SELECT query, and verify the returned result.
#customer intent: As a Node.js developer, I want to connect to Azure Database for PostgreSQL flexible server, so that I can query my database from an application.
author: gkasar
ms.author: gkasar
ms.reviewer: maghan
ms.date: 08/24/2026
ms.service: azure-database-postgresql
ms.subservice: connectivity
ms.topic: quickstart
ai-usage: ai-generated
ms.custom:
  - mvc
  - devx-track-js
  - mode-api
ms.devlang: javascript
---

# Quickstart: Connect and query Azure Database for PostgreSQL with Node.js

In this quickstart, you use Node.js and the `pg` package to connect over Transport Layer Security (TLS) to an existing Azure Database for PostgreSQL flexible server and run a `SELECT` query. The printed database name confirms that the connection and query succeeded.

The same application works on macOS, Linux, and Windows. This quickstart creates only a local Node.js project and uses your existing server; it doesn't provision or change Azure resources.

## Prerequisites

- An Azure account with an active subscription. [Create an Azure account for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- An existing Azure Database for PostgreSQL flexible server. If you need a server, follow [Quickstart: Create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md) before you continue.
- The server host name, database name, user name, and password. Find the server name and administrator login on the server's **Overview** page in the [Azure portal](https://portal.azure.com).
- Network access from your development computer to the server. For a public-access server, allow your client IP address through the server firewall. For a private-access server, run the client from a resource that can reach the server's virtual network. For setup instructions, see [Networking with Azure Database for PostgreSQL flexible server](../network/how-to-networking.md).
- [Node.js and npm](https://nodejs.org/en/download) installed on macOS, Linux, or Windows.
- Trusted root certificates available to the Node.js runtime so that it can verify the server certificate. Review [Configure TLS connections](../security/security-tls-how-to-connect.md) and complete the client certificate setup that applies to your operating system.

## Create the Node.js project

Create a local project and install `pg`, the PostgreSQL client package for Node.js.

1. Open a terminal and create a project folder:

   ```console
   mkdir nodejs-postgresql
   cd nodejs-postgresql
   ```

1. Initialize the project and install `pg`:

   ```console
   npm init -y
   npm install pg
   ```

1. Confirm that npm lists `pg` as an installed dependency:

   ```console
   npm list pg
   ```

   Continue when the output includes `pg` and its installed version.

## Set the connection values

Supply the connection values as environment variables so that the JavaScript file doesn't contain your credentials. Don't commit credentials to source control.

1. In Bash on macOS or Linux, replace each example value with the value for your server, and then run:

   ```bash
   export PGHOST="your-server.postgres.database.azure.com"
   export PGDATABASE="your-database"
   export PGUSER="your-user"
   export PGPASSWORD="your-password"
   export PGPORT="5432"
   ```

1. In PowerShell on Windows, replace each example value with the value for your server, and then run:

   ```powershell
   $env:PGHOST = "your-server.postgres.database.azure.com"
   $env:PGDATABASE = "your-database"
   $env:PGUSER = "your-user"
   $env:PGPASSWORD = "your-password"
   $env:PGPORT = "5432"
   ```

1. Keep this terminal open. Environment variables set by these commands are available to the Node.js process started from the same terminal.

## Create the connection application

Create an application that requires TLS certificate verification, connects with `pg.Client`, and returns the current database name.

1. In the `nodejs-postgresql` folder, create a file named `index.js`.

1. Add the following JavaScript:

   ```javascript
   const pg = require('pg');

   const client = new pg.Client({
     host: process.env.PGHOST,
     database: process.env.PGDATABASE,
     user: process.env.PGUSER,
     password: process.env.PGPASSWORD,
     port: Number(process.env.PGPORT),
     ssl: {
       rejectUnauthorized: true,
     },
   });

   async function main() {
     let connected = false;

     try {
       await client.connect();
       connected = true;

       const result = await client.query(
         'SELECT current_database() AS database_name'
       );
       console.log(`Connected to database: ${result.rows[0].database_name}`);
     } catch (error) {
       console.error('Connection or query failed:', error);
       process.exitCode = 1;
     } finally {
       if (connected) {
         await client.end();
       }
     }
   }

   main();
   ```

1. Save `index.js`.

## Run and verify the query

Run the application from the terminal where you set the connection environment variables.

1. Run the application:

   ```console
   node index.js
   ```

1. Confirm that the output contains the database name you supplied in `PGDATABASE`:

   ```output
   Connected to database: <your-database>
   ```

   This output confirms that `pg` connected over TLS and returned the `SELECT` result. If the application instead prints `Connection or query failed`, check the connection values, network access, and trusted root certificate setup before you retry.

## Clean up resources

The quickstart creates a local project but doesn't create an Azure Database for PostgreSQL flexible server.

1. Keep the `nodejs-postgresql` folder if you want to build on the sample. Otherwise, delete the folder and clear the connection environment variables from the current terminal session.
1. Keep the existing flexible server if other applications use it. If you no longer need the Azure resources, delete only the server or delete its resource group to remove the server and related resources.

## Related content

- [TLS parameters](../parameters/parameters-tls.md)
- [Connection libraries in Azure Database for PostgreSQL flexible server](concepts-connection-libraries.md)
- [Manage Azure Database for PostgreSQL flexible server by using the Azure portal](../configure-maintain/how-to-manage-server-portal.md)
