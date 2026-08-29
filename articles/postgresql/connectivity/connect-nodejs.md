---
title: "Connect to Azure Database for PostgreSQL with Node.js"
description: Connect securely to Azure Database for PostgreSQL with Node.js and pg, run a SELECT query, print the result, and close the client.
#customer intent: As a Node.js developer, I want to connect to Azure Database for PostgreSQL, so that I can query data from my application.
author: gkasar
ms.author: gkasar
ms.reviewer: maghan
ms.date: 08/28/2026
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

# Quickstart: Connect to Azure Database for PostgreSQL with Node.js

Azure Database for PostgreSQL flexible server is a managed PostgreSQL service that applications can connect to and query. The `pg` package provides a Node.js client for this task.

In this quickstart, you install `pg`, connect to an existing Azure Database for PostgreSQL flexible server over Transport Layer Security (TLS), run a simple `SELECT` statement, print the returned value, and close the connection. A printed value of `1` confirms that the query succeeded.

## Prerequisites

- An Azure account with an active subscription. If you don't have an account, [create one for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- An existing Azure Database for PostgreSQL flexible server and its server name, database name, administrator username, and password. If you need a server, [create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md).
- Network access from your Node.js environment to the server. For a server that uses private access, run the application from a resource in the same virtual network. For other connectivity configurations, follow the [Azure Database for PostgreSQL networking guidance](../network/how-to-networking.md).
- Node.js and npm installed in the environment from which you connect.

## Create a Node.js project

Create a project folder and install the `pg` package that the application uses to connect to PostgreSQL.

1. Create a folder named *postgresql-nodejs* and change to that folder.

   ```bash
   mkdir postgresql-nodejs
   cd postgresql-nodejs
   ```

1. Create a default *package.json* file.

   ```bash
   npm init -y
   ```

1. Install `pg`.

   ```bash
   npm install pg
   ```

   Confirm that the command completes without an npm error before you continue.

## Set the connection environment variables

Keep the server password and other connection values out of the source file by setting them as environment variables. Replace each example value with the corresponding value for your flexible server.

1. Set the host, port, database, username, and password in the shell where you'll run the application.

   ```bash
   export PGHOST="myserver.postgres.database.azure.com"
   export PGPORT="5432"
   export PGDATABASE="postgres"
   export PGUSER="myadmin"
   export PGPASSWORD="your-password"
   ```

1. Confirm that all five environment variables are defined in the current shell. Don't print `PGPASSWORD` to the terminal or store it in source control.

## Connect and run a SELECT query

The application creates a `pg.Client` from the environment variables and explicitly enables TLS certificate validation. It then connects, runs `SELECT 1`, prints the returned row, and closes the client.

1. Create a file named *index.js* in the *postgresql-nodejs* folder, and add the following code:

   ```javascript
   const pg = require('pg');

   const client = new pg.Client({
     host: process.env.PGHOST,
     port: Number(process.env.PGPORT || 5432),
     database: process.env.PGDATABASE,
     user: process.env.PGUSER,
     password: process.env.PGPASSWORD,
     ssl: { rejectUnauthorized: true },
   });

   async function main() {
     await client.connect();

     try {
       const result = await client.query('SELECT 1 AS connection_result');
       console.log(result.rows[0]);
     } finally {
       await client.end();
     }
   }

   main().catch((error) => {
     console.error(error);
     process.exitCode = 1;
   });
   ```

1. Run the application from the *postgresql-nodejs* folder.

   ```bash
   node index.js
   ```

1. Confirm that the output contains `connection_result` with the value `1`. If the application reports a connection or certificate error, verify the environment variable values, network access, and client certificate configuration before you retry.

## Clean up resources

This quickstart creates only the local *postgresql-nodejs* project folder. Delete that folder if you don't want to keep the sample. The existing Azure Database for PostgreSQL flexible server isn't changed by the `SELECT` query.

## Next step

> [!div class="nextstepaction"]
> [Configure TLS certificate validation for Azure Database for PostgreSQL](../security/security-tls-how-to-connect.md)
