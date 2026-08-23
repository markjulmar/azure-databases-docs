---
title: "Quickstart: Use Node.js to connect and query data in Azure Database for PostgreSQL flexible server"
description: Connect a Node.js application to Azure Database for PostgreSQL with pg, require TLS, run a SELECT query, and verify the result.
#customer intent: As a Node.js developer, I want to connect to Azure Database for PostgreSQL, so that I can query my database from an application.
author: gkasar
ms.author: gkasar
ms.reviewer: maghan
ms.date: 08/23/2026
ms.service: azure-database-postgresql
ms.subservice: connectivity
ms.topic: quickstart
ai-usage: ai-generated
ms.devlang: javascript
---

# Quickstart: Use Node.js to connect and query data in Azure Database for PostgreSQL flexible server

In this quickstart, you use the `pg` package to connect a local Node.js application to an Azure Database for PostgreSQL flexible server. The connection requires Transport Layer Security (TLS), and the application reads its connection values from environment variables instead of source code.

You run a `SELECT` statement that returns the current database and user. The application prints those values in the console so that you can verify the connection and query succeeded.

## Prerequisites

Before you begin, make sure you have:

- An Azure account with an active subscription. If you don't have one, [create an Azure account for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- An Azure Database for PostgreSQL flexible server. If you need a server, [create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md).
- Network access from your client:
  - For public access, [add your current client IP address to the server's firewall rules](../network/how-to-networking-servers-deployed-public-access-add-firewall-rules.md).
  - For private access, run the application from a resource in the same virtual network as the server.
- Node.js and npm installed on your computer.
- The server host, database name, user name, and password. In the Azure portal, find the server host and administrator user on the server's **Overview** page. You can use the default `postgres` database or another database that your user can access.

For more information about `pg` and other supported clients, see [Connection libraries in Azure Database for PostgreSQL flexible server](concepts-connection-libraries.md).

## Prepare the Node.js project

Create a local project folder, and install `pg` as the only database client library for this quickstart.

1. Open a terminal, and create a project folder:

   ```bash
   mkdir nodejs-postgresql
   cd nodejs-postgresql
   ```

1. Install `pg`:

   ```bash
   npm install pg
   ```

1. Confirm that npm lists `pg` in the project:

   ```bash
   npm list pg
   ```

## Set the connection environment variables

Store the connection values in environment variables so that the JavaScript file doesn't contain your password. Replace each placeholder with the value for your server.

1. Set the environment variables in the terminal where you plan to run the application.

   ### [Windows](#tab/windows)

   ```cmd
   set AZURE_POSTGRESQL_HOST=<server-name>.postgres.database.azure.com
   set AZURE_POSTGRESQL_DATABASE=<database-name>
   set AZURE_POSTGRESQL_USER=<username>
   set AZURE_POSTGRESQL_PASSWORD=<password>
   set AZURE_POSTGRESQL_PORT=5432
   ```

   ### [macOS/Linux](#tab/macos-linux)

   ```bash
   export AZURE_POSTGRESQL_HOST=<server-name>.postgres.database.azure.com
   export AZURE_POSTGRESQL_DATABASE=<database-name>
   export AZURE_POSTGRESQL_USER=<username>
   export AZURE_POSTGRESQL_PASSWORD=<password>
   export AZURE_POSTGRESQL_PORT=5432
   ```

   ---

   Use these values:

   - `<server-name>` is the name of your flexible server.
   - `<database-name>` is `postgres` or another database that your user can access.
   - `<username>` is the administrator or database user name.
   - `<password>` is the password for that user.

1. Keep this terminal open. The application reads the variables from this terminal session.

## Create the connection application

The application creates a `pg.Client`, requires certificate validation for the TLS connection, and queries the current database and user.

1. In the *nodejs-postgresql* folder, create a file named *index.js*.

1. Add the following code to *index.js*:

   ```javascript
   const { Client } = require('pg');

   const client = new Client({
     host: process.env.AZURE_POSTGRESQL_HOST,
     database: process.env.AZURE_POSTGRESQL_DATABASE,
     user: process.env.AZURE_POSTGRESQL_USER,
     password: process.env.AZURE_POSTGRESQL_PASSWORD,
     port: Number(process.env.AZURE_POSTGRESQL_PORT || 5432),
     ssl: { rejectUnauthorized: true },
   });

   async function queryDatabase() {
     try {
       await client.connect();
       const result = await client.query(
         'SELECT current_database() AS database_name, current_user'
       );
       const row = result.rows[0];
       console.log(`Connected to ${row.database_name} as ${row.current_user}.`);
     } finally {
       await client.end();
     }
   }

   queryDatabase().catch((error) => {
     console.error('Connection, query, or shutdown failed:', error);
     process.exitCode = 1;
   });
   ```

1. Save *index.js*. Don't add the password or other connection values directly to this file.

## Run the application and verify the result

Run the application from the terminal that contains your connection environment variables.

1. From the *nodejs-postgresql* folder, run:

   ```bash
   node index.js
   ```

1. Confirm that the console shows the database and user returned by the `SELECT` statement. The output resembles:

   ```output
   Connected to <database-name> as <username>.
   ```

   The database and user in the output should match the values you configured. If the application reports an error, confirm that the environment variables are set and that your client has network access to the server, and then run the command again.

## Clean up resources

The application creates only local project files. When you no longer need the sample, close the terminal to remove its session environment variables, and then delete the *nodejs-postgresql* folder.

If you created an Azure Database for PostgreSQL flexible server only for this quickstart, follow the cleanup guidance in [Create an Azure Database for PostgreSQL flexible server](../configure-maintain/quickstart-create-server.md#clean-up-resources). If its resource group contains resources you want to keep, delete only the flexible server.

## Next step

> [!div class="nextstepaction"]
> [Explore connection and query options](how-to-connect-query-guide.md)
